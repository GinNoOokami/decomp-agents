---
name: ffx-split
description: Splitting FFX (PS2) code into translation units in ~/dev/ffx - choosing what to split, finding file edges (reference projects, .rodata alignment, static .bss, shared literals, saved boundary signals, Switch order, other builds), deciding provisional edges, writing and scratch-building the yaml/C change, recording the evidence, and measuring how much is split or matched. Use when asked to split a range or system into files, find or check a file boundary/edge, propose yaml segments, judge a reference project's TU edge, or report split/match progress.
---

# FFX split

A split gives a range of the main ELF its own C file (`INCLUDE_ASM` stubs until matched) and, where
the evidence allows, its own `.rodata`. Edges are hypotheses: the yaml makes them cheap to move
while the functions are still asm, and matched C at a wrong edge has to move files. So say how
strong each edge is, and prefer edges that serve the goal over edges that are merely clean.

Reference files: `queries.md` (target query, edge-signal queries, khasinski conversion),
`applying.md` (yaml patterns, splat behaviour, scratch build, after the split).
Tools: `~/dev/ffx-analysis/scripts/` (index `~/dev/DecompNotes/FFX/Tools.md`); run them as
`python3 scripts/X.py` (they aren't executable). Edge facts and
past splits: `~/dev/DecompNotes/FFX/FileBoundaries.md`; system ranges: `FFX/SystemSplit.md`.

## Working agreement

- **Propose before editing the user's tree.** Show the edges with their evidence and the yaml
  change; apply only on "apply"/"go ahead" (re-read the yaml first; check `git status` and
  `pgrep -a make`; back up the yaml to scratch).
- **Every proposed change is built first** in the scratch repo (`applying.md`); never hand over an
  untested yaml.
- **The target comes from the goal**, not from the cleanest edge: files holding functions matched C
  already calls (prototypes in `include/extern.h` and the area headers such as `game/battle.h`),
  or tables/functions the user wants in C. Rank candidates by call sites from the split files'
  asm (`queries.md` "Target"), not by prototype counts alone. Start from that cluster; widen only
  as far as the evidence forces.
- **Pilot first, report the yield.** One region, cheapest checks, then decide whether more is worth
  it. No new framework for what a sqlite query or grep does.
- **Edges are leads until matched C confirms the file**; reference projects corroborate, never
  replace, our evidence. In notes: observation, then inference, labelled.

## Procedure

1. **Notes:** `notescheck.py` on every address and name involved; quote what the notes have and
   keep "already noted" apart from "new" in the report. Its "inside a range" lines repeat for
   every address: keep the direct hits (`| head`, or one address at a time).
2. **Reference projects** (leads):
   - khasinski `tu_map.tsv` (PAL addresses, exclusive ends; conversion and grep recipe in
     `queries.md`). Note its confidence column; its edges are statistical.
   - Rozelette's yaml/src (`ref/rozelette-ffx/SLPS_250.88.yaml`), Sony sources in `ref/globe`.
3. **Switch order:** `swrun.py FUNC --anchor SwitchName -n 12` around each candidate edge; Switch
   name runs in `builds/switch/out/functions.tsv` (grep a prefix). Order shows which functions sit
   together; it can't place an edge (the port moves blocks, e.g. the `MsBtlRead*` block sits
   between `MsGetDamage*` and `MsGetRom*` on Switch). `swfill.py` far from its anchors (`edgeN`
   with big N) is unreliable. Rows drift where PS2 and Switch order a pair differently (1B3FC8:
   PS2 has the free worker 0x1B4218 before `MsBtlChrFreeAll` 0x1B4350, Switch after it); when sizes
   stop tracking, settle the pair by callers (`swfunc.py --callers` vs `symuse.py`) before
   trusting a row.
4. **Edge signals** — the checklist below, cheapest first.
5. **Decide:** each edge = the narrowest window the signals allow, then a chosen point inside it
   with its reason (goal, Switch order, reference edge). Inside the window, take the narrowest
   edge that covers the goal functions, plus callers a worker needs to become `static`: an edge
   too narrow costs a yaml change and appended `INCLUDE_ASM`s later, one too wide means moving
   matched C. Say which edges rest on our own signals
   and which only on a reference project.
6. **Build in scratch, propose, apply** (`applying.md`). The proposal shows the checklist with
   each check marked ran / no signal / skipped.
7. **Record** the evidence in FileBoundaries.md (a short section per split: a 1–2 line contents
   summary, window, signals, rejected reference edges, open ends; commit + push), and report progress with
   `python3 scripts/layout.py count LO HI` (functions, bytes, in C files, matched, matched %).

## Edge checklist

Each gives a datapoint; the cut rests on the ones that hold. Facts behind each test (probes,
calibration) are in FileBoundaries.md "Signals and how far they hold".

| # | Check | How | Reads as |
|---|---|---|---|
| 1 | Saved signals | `combine.py cuts LO HI`, `combine.py files LO HI`; `dataorder.py gaps LO HI` | exact cut / joined run; discount one-pair `.data` joins into big shared blocks (`btl`, `EventSave` fields) |
| 2 | `.rodata` alignment | `.rodata` items + users (query below) at each block start | a string at a 16-aligned address that skips a free 8-aligned slot after the previous item = new object. Jump tables are `.align 4` (16) inside one object too: padding after a table is no signal. No signal either when the next free 8-aligned slot is itself 16-aligned |
| 3 | Static `.bss` | users of each `.bss` item near the edge (query below) | in the low block 0x57C780–0x6A2C90, two functions using one item = same file (ee-gcc puts only uninitialized statics there; `= 0` goes to `.data`, plain globals to COMMON). Also: where the previous/next file's items' users stop. One file's low `.bss` right before the next split file's only means the files between have no statics, not that the two files are adjacent |
| 4 | Shared literals | same `.rodata` address used by two functions | same file (gcc 2.9 merges identical literals per TU only); two copies of one string = different files |
| 5 | Date getters | AlphaBuild.md list | which side of the edge, by the alignment test |
| 6 | Call direction | `calldir.py gaps LO HI`; callers per function (`symuse.py FUNC`) | leads only; callers only inside the file suggests `static` |
| 7 | Other builds | `xmap.py lookup ADDR` at both sides | a function missing in an older build, or alpha-only code International dropped, often sits at a file end; a delta step inside a run is an insertion, not an edge |
| 8 | Per-object `.data` blocks | `.data` users per block (`symuse.py ADDR --size --range`) | gcc emits one `.data` block per object: far-apart blocks used by neighbouring functions = separate files (TK layer, 2026-11-10); high `.bss` blocks with no outside users are weak alone |

Queries for checks 2–4: `queries.md` "Edge signals".

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
