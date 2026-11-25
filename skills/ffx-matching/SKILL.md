---
name: ffx-matching
description: Matching FFX (PS2) functions byte-for-byte in ~/dev/ffx with ee-gcc 2.9. Working agreement, procedure, and verified codegen lessons indexed by symptom. Use when matching, giving matching hints, reviewing a translated function, deciding types, struct layouts or global declarations from asm, or stuck on a register, order, layout or size difference. Cleanup of matched code (names, comments, constants, declaration placement) is the ffx-cleanup skill.
---

# FFX matching

Toolchain: ee-gcc 2.9-991111 (gcc 2.95.2 based, haifa scheduler), `-O2 -G0 -fno-strict-aliasing`;
two assemblers chosen per source folder (Makefile `DTLS_DIRS`: dtls13010 for those, 991111-01 for the
rest). gcc pass sources:
`~/dev/ffx-analysis/gcc/gcc-2.95.2-ps2linux-3a/gcc/`. Tool locations: `~/dev/DecompNotes/FFX/Tools.md`.
Problems the user hits repeatedly are in `~/dev/DecompNotes/FFX/Troubleshooting.md` (cited here as T<n>).

## Working agreement (always applies)

- **The user does the matching.** Give hints, not code: the smallest pointer that unblocks (a type, a
  branch shape, an asm detail). A private scratch match is fine for checking a hint; never quote it
  unless asked for a full match.
- **Verify before hinting.** Compile the hint in scratch first, or prove it from the asm, accounting
  for delay slots. Label anything untested "Untested guess:". Hints taken from a scratch match must be
  applied literally to the user's *current* file and confirmed; quote the exact form when form matters
  (func_00159EA8: "keep the result in a local" lost the chained `G = tmp = call()` the match needed).
  An expression simplified for explanation is "for reading only" unless compiled (an explained
  `x & ~mask` got used as code and cost a `nor`). When several variants tie, report the simplest one,
  closest to the user's code. Offer the next hint rather than the answer.
- **"Matches" = full build, sha1 OK**, in the scratch repo (`scratch.py --make`). variant.py MATCH
  checks named relocations by address, but not section-relative ones (string literals, jump tables)
  or unresolved names (its summary says how many), and it never sees .rodata/.data layout.
- **Hint requests** ("tips", "suggestions", "hints"): do the procedure's first two steps anyway
  (target `.s`, Switch/PC counterparts, refs), then give the one hint that unblocks the first problem,
  and offer the next instead of listing everything.
- **Two phases.** While the user is turning m2c into C, review behaviour and control flow only;
  register, stack-slot and block-placement differences wait until they say they are matching. Say
  which kind of check was run (logic read, scratch compile, full build).
- **"Did I fix X?" reviews** diff the whole function in scratch and report every other structural
  difference too; never call an m2c artifact (an extra guard, a leftover tail) harmless without
  compiling it.
- **Hands off the user's work.** Never edit the function they are working on; work on scratch copies.
  Exceptions only when asked: parking (stuck.md) and "apply X" (re-read their file first). A full
  match they ask for ("match it for me") is delivered for the user to apply (the function plus any
  config edits, in the agent's usual copy format); the tree is edited only on "apply".
- **No fake matches.** A `goto`, an empty `do {} while (0)`, a local holding a constant, permuter
  oddities: diagnostics that say "the structure differs here", never progress. This holds even when
  such a version matches: the one or two matched functions with gotos (func_001632B8) only mean the
  natural shape hasn't been found; there is no hard evidence the original used gotos, so keep looking
  for the natural shape. Status for such a match: "goto match, natural shape not found, N structured
  shapes ruled out". Hard evidence would be a pass-rule proof from the gcc source that no structured
  construct gives the target's RTL, each candidate shape failing for a stated reason. C nobody would
  write means wrong types or layout (see types-and-shapes.md).
- **"Unmatched" claims come from the source:** grep INCLUDE_ASM vs C before calling a function open.
  An unnamed "this function" is the one in the user's git diff.
- **Report progress as structure:** what target detail is now explained, what is ruled out, whether
  the shape is plausible source. Score alone is not progress (stuck.md).
- **"What's left" lists come from a diff** of the user's function against the verified copy,
  leave-one-out tested, never from recall.

## Procedure

1. **Start.** (`nextfunc.py lib/<tu>` picks a target when none is given.) Copy the user's current
   file to scratch and work from their version, reporting results as changes against it. Copy the
   target `asm/jp_intl/nonmatchings/<unit>/<func>.s` to scratch now: splat only writes `.s` for
   INCLUDE_ASM functions, so it vanishes on the next re-split (yaml or symbol_addrs change,
   `make distclean`). Before applying a scratch result, `cmp` the tree file against the copy's base;
   if the user edited it meanwhile, re-apply onto their current file.
2. **Gather counterparts before guessing:** Switch (`swfunc.py FUNC`; with a stale or missing row, the
   full list in `builds/switch/out/functions.tsv` + `swfunc.py --sw`, never `swx2/shared.tsv`), PC
   (`builds/pc/out/`), the ref
   repos (khasinski `tu_map.tsv`, Rozelette, `ref/globe` Sony sources), matched siblings in the TU.
   PC keeps helper boundaries that Switch inlines: an inline's real signature comes from PC. Find a
   PC body through a known PC caller's callees (TkScanControler's counterpart calls 0x889940 twice).
   Always do the Switch lookup (it is also the naming evidence); stop gathering the rest once one
   counterpart pins the shape or the first compile matches. Parking still needs every source
   covered (stuck.md).
3. **Look up the symptom** in the index below and in Troubleshooting.md's index.
4. **Compile only through the tools:** `variant.py FILE FUNC -f snippet.c` (`--repo` when headers
   change). Check Tools.md before writing any script; extend a tool rather than add a one-off.
   `scratch.py` syncs the user's WIP as is; a replacement definition plus config edits go in one
   call (`--fn F.c --func F -e e.py --src … --make`: the definition first, then the edits).
5. **Near-match:** `vsweep.py SRC FUNC pack` on *every* new best version (levers interact), then
   `variant.py --proto` for prototype tests, `rtl.py --diff A.c B.c` for "why does B fix it",
   `rtl.py --priority` for register choices.
6. **After two or three failed hand-written variants on one detail** (tool sweeps like `pack` and
   `orders` are a cheap one-shot and don't count), stop guessing: trace the pass in
   the RTL dumps and gcc source, work out which source shapes satisfy it, test only those. Reading one
   pass is cheaper than many builds and answers *why* (func_00163950 / func_00255158: ~15 variants
   over a session, 83-90%, none natural, gcc source never opened).
7. **Stubborn:** stuck.md (cover every idea source, tried tables, parking).
8. **After a match:** full build (never while the user is building: `pgrep make` first). Once the
   match is in the tree (not for one that exists only in scratch): offer `tidy_externs.py`; rename
   only matched functions, with the user picking the name; log the harder/easier/same rating
   against a recent non-twin match in `~/dev/ffx-analysis/data/nextfunc_ratings.tsv`.

## Symptom index

| Symptom | File |
|---|---|
| Register swap, wrong saved register, extra spill/stack slot, frame size | regalloc.md |
| Same instructions in a different order; load/store order; hoisting | order-and-scheduling.md |
| Exits, tail calls, switch dispatch, loop shape, branch layout, merged/duplicated tails | control-flow.md |
| Masks, signedness, bitfields, casts, reassociation, struct/array shapes, globals | types-and-shapes.md |
| m2c output that misleads (arguments, callee params, inlined copies) | m2c-traps.md |
| Link/size errors, .rodata/.bss ownership, delay slot only, assembler, inline asm | build-and-asm.md |
| How to compile, compare, probe and trust tool results | tools-and-verification.md |
| Stuck for several rounds; parking | stuck.md |

Function names in these files are evidence (where the lesson was verified); some may have been
renamed since, so grep `symbol_addrs.txt` by address too.
