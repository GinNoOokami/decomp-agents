---
name: ffx-split
description: Splitting FFX (PS2) code into translation units in ~/dev/ffx - choosing what to split, finding file edges (reference projects, .rodata alignment, static .bss, shared literals, saved boundary signals, Switch order, other builds), deciding provisional edges, writing and scratch-building the yaml/C change, recording the evidence, and measuring how much is split or matched. Use when asked to split a range or system into files, find or check a file boundary/edge, propose yaml segments, judge a reference project's TU edge, or report split/match progress.
---

# FFX split

A split gives a range of the main ELF its own C file (`INCLUDE_ASM` stubs until matched) and, where
the evidence allows, its own `.rodata`. Edges are hypotheses: the yaml makes them cheap to move
while the functions are still asm, and matched C at a wrong edge has to move files. So say how
strong each edge is, and prefer edges that serve the goal over edges that are merely clean.

Reference file: `applying.md` (yaml patterns, splat behaviour, scratch build, after the split).
Tools: `~/dev/ffx-analysis/scripts/` (index `~/dev/DecompNotes/FFX/Tools.md`). Edge facts and
past splits: `~/dev/DecompNotes/FFX/FileBoundaries.md`; system ranges: `FFX/SystemSplit.md`.

## Working agreement

- **Propose before editing the user's tree.** Show the edges with their evidence and the yaml
  change; apply only on "apply"/"go ahead" (re-read the yaml first; check `git status` and
  `pgrep -a make`; back up the yaml to scratch).
- **Every proposed change is built first** in the scratch repo (`applying.md`); never hand over an
  untested yaml.
- **The target comes from the goal**, not from the cleanest edge: files holding functions matched C
  already calls (`include/extern.h` declarations), or tables/functions the user wants in C. Start
  from that cluster; widen only as far as the evidence forces.
- **Pilot first, report the yield.** One region, cheapest checks, then decide whether more is worth
  it. No new framework for what a sqlite query or grep does.
- **Edges are leads until matched C confirms the file**; reference projects corroborate, never
  replace, our evidence. In notes: observation, then inference, labelled.

## Procedure

1. **Notes:** `notescheck.py` on every address and name involved; quote what the notes have and
   keep "already noted" apart from "new" in the report.
2. **Reference projects** (leads):
   - khasinski `~/dev/DecompNotes/ref/khasinski-ffx/config/tu_map.tsv` (PAL addresses; the offset
     to ours varies by region: take it from `xmap.py lookup ADDR` near the edge, e.g. +0x138 in
     battle core 0x1B8920–0x1C05E8). Note its confidence column; its edges are statistical.
   - Rozelette's yaml/src (`ref/rozelette-ffx/SLPS_250.88.yaml`), Sony sources in `ref/globe`.
3. **Switch order:** `swrun.py FUNC --anchor SwitchName -n 16` around each candidate edge; Switch
   name runs in `builds/switch/out/functions.tsv` (grep a prefix). Order shows which functions sit
   together; it can't place an edge (the port moves blocks, e.g. the `MsBtlRead*` block sits
   between `MsGetDamage*` and `MsGetRom*` on Switch). `swfill.py` far from its anchors (`edgeN`
   with big N) is unreliable.
4. **Edge signals** — the checklist below, cheapest first.
5. **Decide:** each edge = the narrowest window the signals allow, then a chosen point inside it
   with its reason (goal, Switch order, reference edge). Say which edges rest on our own signals
   and which only on a reference project.
6. **Build in scratch, propose, apply** (`applying.md`).
7. **Record** the evidence in FileBoundaries.md (a short section per split: window, signals,
   rejected reference edges, open ends; commit + push), and report progress with
   `layout.py count LO HI` (functions, bytes, in C files, matched, matched %).

## Edge checklist

Each gives a datapoint; the cut rests on the ones that hold. Facts behind each test (probes,
calibration) are in FileBoundaries.md "Signals and how far they hold".

| # | Check | How | Reads as |
|---|---|---|---|
| 1 | Saved signals | `combine.py cuts LO HI`, `combine.py files LO HI`; `dataorder.py gaps LO HI` | exact cut / joined run; discount one-pair `.data` joins into big shared blocks (`btl`, `EventSave` fields) |
| 2 | `.rodata` alignment | `.rodata` items + users (query below) at each block start | a string at a 16-aligned address that skips a free 8-aligned slot after the previous item = new object. Jump tables are `.align 4` (16) inside one object too: padding after a table is no signal |
| 3 | Static `.bss` | users of each `.bss` item near the edge (query below) | in the low block 0x57C780–0x6A2C90, two functions using one item = same file (ee-gcc puts only uninitialized statics there; `= 0` goes to `.data`, plain globals to COMMON). Also: where the previous/next file's items' users stop |
| 4 | Shared literals | same `.rodata` address used by two functions | same file (gcc 2.9 merges identical literals per TU only); two copies of one string = different files |
| 5 | Date getters | AlphaBuild.md list | which side of the edge, by the alignment test |
| 6 | Call direction | `calldir.py gaps LO HI`; callers per function (`symuse.py FUNC`) | leads only; callers only inside the file suggests `static` |
| 7 | Other builds | `xmap.py lookup ADDR` at both sides | a function missing in an older build, or alpha-only code International dropped, often sits at a file end; a delta step inside a run is an insertion, not an edge |
| 8 | Per-object `.data` blocks | `.data` users per block (`symuse.py ADDR --size --range`) | gcc emits one `.data` block per object: far-apart blocks used by neighbouring functions = separate files (TK layer, 2026-11-10); high `.bss` blocks with no outside users are weak alone |

Queries on the layout db (`DB=$(layout.py db)`; addresses are vram):

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

Spot-check a decisive join in the asm (`grep` the `D_` symbol in `asm/jp_intl/*.s`) before
relying on it.

## Lessons (with the split that taught them)

- **Static `.bss` beats reference edges** (1B8920, 2026-11-26): `D_00662080` used by 0x1B8C30 and
  0x1BBCC0 rejected two khasinski edges inside that span.
- **`.rodata` skip gives a window, not a point** (1B8920): the skip only says the new file's first
  `.rodata` user is after the old file's last; functions in between with no data can go either
  way. Choose by goal and say so.
- **No pad by default** (1B8920/1BC308): an INCLUDE_ASM jump table's `.s` carries its trailing zero
  words, so the C object's `.rodata` already ends 16-aligned. Build without a `pad` first; add one
  only when the build shifts (the 1AF4E8 case: matching the last function drops that padding).
- **Read the checklist before the pilot**, not after (1B8920: four checks were run only after the
  proposal; none changed it, but the order matters when one does).
