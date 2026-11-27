# Applying a split

## yaml (`config/jp_intl/SLPS_250.88.yaml`)

Segment offsets are file offsets (vram − 0x100000). A split inside an asm segment adds the C
segment(s) and restarts asm after them:

```yaml
    - [0x0B34F8, asm, 1B34F8]
    - [0x0B8920, c, game/battle/1B8920]
    - [0x0BC308, c, game/battle/1BC308]
    - [0x0C05E8, asm, 1C05E8]
```

`.rodata` end with no skip (the last item runs straight into the next at an aligned address):
end = the first item whose user lies past the file's `.text` end (1CCAF8: 0x56E580, 0x1CFA08's
jump table).

Give the file its `.rodata` when the block's edges are known (needed when its functions own jump
tables: an asm `.rodata` file can't resolve the `.L` labels of a function that moved into a C
object):

```yaml
    - [0x46DDE0, rodata, 56DDE0]
    - [0x46E030, .rodata, game/battle/1B8920]
    - [0x46E100, .rodata, game/battle/1BC308]
    - [0x46E1B0, rodata, 56E1B0]
```

- Names: address-named files in our own folders (`game/battle/`, `game/atel/`, `common/`, `lib/`);
  each segment name distinct.
- Folder: by default the SystemSplit.md block the address is in (link order), not who calls it.
  When callers point elsewhere (1DBA28: getters called mostly from menu code at 0x22/0x24xxxx),
  say so in the proposal and let the user choose. While every function is still `INCLUDE_ASM`, a
  folder change costs only the yaml plus `git mv`.
- `pad` (`- [0x46DDD8, pad, 56DDD8] # the .rodata after it starts 16-aligned; subalign 8 can't
  express that`) only when the build shows a shift; test without first.
- `.data`/`.bss` dot-subsegments hand the range to C, which must then define every variable (exact
  `D_` names while asm still uses them, `size:` for inner labels). Leave data in asm at a first
  split; that belongs to the per-file pass.
- No colons inside `symbol_addrs.txt` comments (splat parses `key:value` there).

## What splat does

- `make` re-splits when the yaml changed and **writes the new C file** (`#include "common.h"` +
  one `INCLUDE_ASM` per function) only if it doesn't exist.
- Literals used by several functions get `D_*.s` + `INCLUDE_RODATA` lines in the new file by
  itself (1BC308: `D_0056E128` `"ply_data"`); for an existing file they must be added by hand.
- Functions nothing references get no `.s` in a C segment and silently vanish (everything after
  shifts); address-only callbacks become `.aent`, which the old `as` rejects. Fix:
  `func_X = 0x…; // type:func` in symbol_addrs plus the `INCLUDE_ASM`.
- Every split may rewrite `include/include_asm.h`, `macro.inc`, `labels.inc`: never hand-format them.
- After a revert of matched C to `INCLUDE_ASM`, a stale `asm/` needs `make distclean build`.

## Scratch build

```sh
python3 scripts/scratch.py -e edits.py   # sync ~/dev/ffx into out/scratchrepo + apply 3-tuple yaml edits
cd ~/dev/ffx-analysis/out/scratchrepo && make 2>&1 | grep -v 'repeat count is zero' | tail
```

Expect `SLPS_250.88.bin: OK`. Then check in `build/jp_intl/SLPS_250.88.map` that each new object's
`.text` and `.rodata` start at the yaml addresses and the next asm segment starts where it should
(`grep -n -E '^ \.(text|rodata) +0x0*<addr>' …map`, lowercase hex). Never build while the user builds (`pgrep -a make`).

## Proposal and after

- Show the yaml lines and what `make` generates (counts of `INCLUDE_ASM`/`INCLUDE_RODATA`), the
  build result, and the edge evidence with its strength, in this shape:

  ```
  Target: <block> — why (ranking: sites / callers / matched-C per candidate)
  Edges:  start <addr> (window <lo>..<hi>, decided by …), end <addr> (…)
  | # | Check              | Result                           |
  | 1 | Saved signals      | ran: … / no signal / skipped (why) |
  | 2 | .rodata alignment  |                                  |
  | 3 | Static .bss        |                                  |
  | 4 | Shared literals    |                                  |
  | 5 | Date getters       |                                  |
  | 6 | Call direction     |                                  |
  | 7 | Other builds       |                                  |
  | 8 | Per-object .data   |                                  |
  | — | notescheck rerun on addresses this report adds |       |
  yaml: …   scratch build: OK / …   folder: … (callers elsewhere?)
  ```
- After applying: full build in the tree, `git status` shows the yaml + new C files; the user
  commits.
- Then: move prototypes for the file's functions from `extern.h` to the area header when they get
  matched (ffx-cleanup), and record the split in FileBoundaries.md.
