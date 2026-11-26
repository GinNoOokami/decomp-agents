# Split queries

All on the layout db: `DB=$(python3 ~/dev/ffx-analysis/scripts/layout.py db)`. Addresses are vram;
yaml offsets are vram − 0x100000. Unsplit = the yaml's `asm` segments.

## Target: what a split file depends on

Calls from a file's range into unsplit code, from the asm (`refs`), not the C: a fresh file's
functions are mostly `INCLUDE_ASM`, so grepping the C misses their calls (1B8920: 35 callers of
`func_001B4150`, almost none visible in C). Matched C's calls are a subset. `kind` `call` = `jal`,
`jump` = tail call `j`. Fill the target ranges from the yaml's asm segments:

```sh
sqlite3 -tabs $DB "select printf('%X',target), count(*), count(distinct func),
 group_concat(distinct printf('%X',func)) from refs
 where func >= 0x1B8920 and func < 0x1BC308 and kind in ('call','jump')
   and ((target >= 0x1B34F8 and target < 0x1B8920) or (target >= 0x1C05E8 and target < 0x1F0000))
 group by target order by 3 desc limit 20"
```

Callee prototypes live in the area headers (`include/game/battle.h`, `chr.h`, `atel.h`) as well as
`include/extern.h`; a declared callee with many call sites is the strongest target.

## Edge signals

```sh
# per function in a range: .rodata and .bss targets
sqlite3 -tabs $DB "select printf('%X',f.addr), f.name, f.size,
 (select group_concat(distinct printf('%X',r.target)) from refs r where r.func=f.addr and r.tsec like '%rodata%'),
 (select group_concat(distinct printf('%X',r.target)) from refs r where r.func=f.addr and r.tsec like '%bss%')
 from functions f where f.addr between 0x1B8000 and 0x1C0800 order by f.addr"
# .rodata / .bss items in a window, and their users
sqlite3 -tabs $DB "select printf('%X',addr), directive, substr(value,1,40) from dataitems where addr between 0x56DF80 and 0x56E1A0 order by addr"
sqlite3 -tabs $DB "select printf('%X',target), group_concat(distinct printf('%X',func)) from refs where target between 0x662000 and 0x662180 and kind in ('hi','gp_rel') group by target"
```

Spot-check a decisive join in the asm (`grep` the `D_` symbol in `asm/jp_intl/*.s`).

## khasinski `tu_map.tsv`

`~/dev/DecompNotes/ref/khasinski-ffx/config/tu_map.tsv`: columns `tu_id first_func last_func
start_addr end_addr name_guess evidence confidence`, PAL addresses (uppercase hex, `0x` prefix).
`end_addr` is exclusive: the end of the last function, before alignment padding.

- Offset: `python3 scripts/xmap.py lookup ADDR` near the edge (`pal_en:` row); it varies by
  region (+0x138 in battle core 0x1B3FC8–0x1C05E8).
- Ours = PAL − offset; an end rounds up to our next function start.
- Rows near ours 0x1B3FC8 (PAL 0x1B4100): `grep -P '\t0x1B4[0-9A-F]{3}\t' tu_map.tsv`.
- `funcs.txt` / `symbol_addrs.txt` beside it give their per-function names (LLM-applied: leads).
