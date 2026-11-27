# Split queries

All on the layout db: `DB=$(python3 ~/dev/ffx-analysis/scripts/layout.py db)`. Addresses are vram;
yaml offsets are vram − 0x100000. Unsplit = the yaml's `asm` segments.

## Target: what a split file depends on

Calls from a file's range into unsplit code, from the asm (`refs`), not the C: a fresh file's
functions are mostly `INCLUDE_ASM`, so grepping the C misses their calls (1B8920: 35 callers of
`func_001B4150`, almost none visible in C). Matched C's calls are a subset. `kind` `call` = `jal`,
`jump` = tail call `j`. The ranges come from the current yaml (the layout db's `segments` table
has only top-level sections and predates recent splits). Set the system range and C-file prefix,
run from `~/dev/ffx-analysis`:

```python
import re, sqlite3, subprocess
LO, HI, PREFIX = 0x1AC068, 0x1EF7D8, "game/battle/"   # system range; C files counted as callers
segs = []
for l in open("/home/jeff/dev/ffx/config/jp_intl/SLPS_250.88.yaml"):
    m = re.match(r"\s*- \[(0x[0-9A-Fa-f]+), (c|asm|hasm|data|\.?rodata|\.?data|pad|bin)\b,? ?([^\]]*)\]", l)
    if m and int(m[1], 16) < 0x200000:                  # .text segments
        segs.append((int(m[1], 16) + 0x100000, m[2], m[3].strip()))
segs.sort()
rng = [(a, segs[i + 1][0], k, n) for i, (a, k, n) in enumerate(segs[:-1])]
inside = lambda col, kinds: " or ".join(f"({col} >= {max(a, LO)} and {col} < {min(b, HI)})"
                                       for a, b, k, n in rng if k in kinds and a < HI and b > LO
                                       and (k != "c" or n.startswith(PREFIX)))
db = sqlite3.connect(subprocess.check_output(["python3", "scripts/layout.py", "db"], text=True).strip())
q = (f"select printf('%X',target), count(*), count(distinct func) from refs where kind in ('call','jump') "
     f"and ({inside('func', {'c'})}) and ({inside('target', {'asm'})}) group by target order by 3 desc, 2 desc limit 20")
for r in db.execute(q):
    print(*r, sep="\t")      # callee, call sites, distinct callers
```

Then group the callees into blocks (SKILL.md "Rank candidate blocks") with the neighbourhood table
below, and add calls from matched C: a call written in C text is matched code (`INCLUDE_ASM` has no
call text), so `grep -rn 'func_001CCA48\|MsStructClear' ~/dev/ffx/src` (old and current name;
matched callees get renamed).

Callee prototypes live in the area headers (`include/game/battle.h`, `chr.h`, `atel.h`) as well as
`include/extern.h`; a declared callee with many call sites is the strongest target.

## Edge signals

```sh
# neighbourhood table: per function, size, distinct callers, .rodata / .bss / .data targets
sqlite3 -tabs $DB "select printf('%X',f.addr), f.name, f.size,
 (select count(distinct r.func) from refs r where r.target=f.addr and r.kind in ('call','jump')),
 (select group_concat(distinct printf('%X',r.target)) from refs r where r.func=f.addr and r.tsec like '%rodata%'),
 (select group_concat(distinct printf('%X',r.target)) from refs r where r.func=f.addr and r.tsec like '%bss%'),
 (select group_concat(distinct printf('%X',r.target)) from refs r where r.func=f.addr and r.tsec = '.data')
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
