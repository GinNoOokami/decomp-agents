# Split queries

`python3 scripts/splitscan.py` (in `~/dev/ffx-analysis`) answers the ranking and edge questions;
it reads the layout db and the current yaml. Addresses are vram (yaml offset + 0x100000).

| Subcommand | Answers | Notes |
|---|---|---|
| `targets LO HI PREFIX` | unsplit callees of the C files under `src/PREFIX` in [LO, HI): call sites, distinct callers, matched-C callers (init/free hubs starred), notes hits; by address, blank line between blocks | ranked by matched-C callers without hubs, then callers, then sites (`--top`, `--gap`) |
| `funcs LO HI` | per function: size, distinct callers, `.rodata`/`.bss`/`.data` targets | the neighbourhood table for grouping and edges |
| `skips LO HI` | each referenced `.rodata` item vs the previous item's end: SKIP (new object), skip into a jump table (no signal), contiguous at 8 (no signal), with users on both sides | reproduces the 1B8920 edge at 0x56E030 |
| `bss LO HI` | `.bss` items in address order with their users | where one file's statics stop and the next start |
| `khasinski ADDR` | khasinski's `tu_map` rows near ADDR in our addresses (PAL offset from `xmap`) | ends are exclusive, before padding: round up to our next function; their edges are statistical |

Matched-C callers come from C text: a call written in C is matched code (`INCLUDE_ASM` has no call
text), found under the callee's current name or its `func_` name.

## Extra queries (layout db)

`DB=$(python3 scripts/layout.py db)`

```sh
# .rodata items in a window with their directive and value (what skips summarizes)
sqlite3 -tabs $DB "select printf('%X',addr), directive, substr(value,1,40) from dataitems where addr between 0x56DF80 and 0x56E1A0 order by addr"
```

Spot-check a decisive join in the asm (`grep` the `D_` symbol in `asm/jp_intl/*.s`).
khasinski's per-function names (`funcs.txt`, `symbol_addrs.txt` beside `tu_map.tsv`) are
LLM-applied: leads.
