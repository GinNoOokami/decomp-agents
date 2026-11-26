---
name: ffx-cleanup
description: Cleaning up matched FFX (PS2) code in ~/dev/ffx: per-function cleanup (applied directly) and the per-file pass once a TU is fully matched (proposed first), checklists, Switch-backed function renames, local/parameter/field/global names, lead comments, constants and defines, declaration placement, and reading-quality code shape. Use when asked for "cleanup" of a function or TU, when proposing names or comments for matched code, or when reviewing the user's cleanup.
---

# FFX cleanup

Cleanup turns a matched function into the source a reader can trust: names and comments that say
what the code shows, constants that say what they mean, declarations where the evidence puts them,
and natural structure. It never changes the match. Codegen questions (why a shape matches, which
levers exist) belong to the `ffx-matching` skill; this skill only uses its results.

Reference files:

| Topic | File |
|---|---|
| Function, parameter, local, field and global names; lead comments for names | naming.md |
| What a comment may say, lead-comment shape, evidence strength, the user's comments | comments.md |
| Prototypes and externs, constants/defines/enums, struct fields and tags | declarations.md |
| Locals, nesting, statement forms, reading quality (overlap with matching) | code-shape.md |

Tools live in `~/dev/ffx-analysis/scripts/` (index: `~/dev/DecompNotes/FFX/Tools.md`).

## Two passes

The workflow is: match → per-function cleanup → repeat until the TU has no `INCLUDE_ASM` →
per-file cleanup (when possible).

| | Per-function | Per-file |
|---|---|---|
| When | right after each match ("cleanup") | the whole TU is matched (no parked nonmatchings: they block moving `.data`/`.rodata` in) |
| Scope | the function and the globals, fields, prototypes it touches | the TU: data ownership, linkage, headers, vocabulary |
| Mode (clean tree) | **apply everything, build, report**; the user raises changes afterwards | **propose a plan, agree it, then apply** |

Per-function cleanup is small and frequent, so a round trip costs more than a later correction.
Per-file cleanup is large and rare, so reworking an applied change costs more than agreeing first.

## Working agreement (always applies)

- **First command of every cleanup turn: `git diff --quiet; echo $?`.** Run it, don't read
  `git status`: `M ` (staged only) counts as clean. Check again right before editing, since the
  user stages WIP around their messages, and **again as the last command before sending a
  proposal**: the user often stages right after sending the prompt, so an early "1" may be stale.
  If it turned 0, apply instead of proposing.
  - **0 (clean), per-function:** apply every item of the procedure in one go, build, report. That
    includes Switch renames (functions with body + position checked; globals via `rename.py --data
    --switch` when Switch uses them like ours, check the size attribute), types the ELF bytes or
    Switch settle (`char[]` + `// "contents"` for strings), leads (`// Switch Name?`, `// name?`),
    our own parameter/local/field names, defines, struct gap splits, compiled tidy-ups, prototypes
    into the existing area header. Don't stop for a "go ahead"; mark guesses in the report.
    Exception: a function without a Switch/original name keeps `func_` and gets a `// Name?` lead
    (the user picks function names).
  - **0 (clean), per-file:** propose the plan (per-file section below); apply only the agreed list.
  - **1 (unstaged changes):** propose everything; the tree is the user's. "Go ahead" / "apply" on
    a proposal is permission for exactly that list; re-read the file first, the user may have
    edited it.
- **Never touch the user's in-progress function** beyond what they asked for, and never undo their
  edits or style (decimal vs hex, their comments; see comments.md).
- **Every edit is checked:** a full build (`make`, filter `grep -v 'repeat count is zero'`, expect
  `OK`) before reporting; never while the user builds (`pgrep -a make`). Comment-only edits need no
  build. Back up a file to scratch before editing it.
- **Code changes compile first.** Every code change (removed local, define, reordered statement)
  is tested in `variant.py SRC FUNC -e edits.py` before it is applied or offered. Renames and
  comments need no test.
- **Report as "I edited" vs "for you to apply"**, against the user's current code. List only what
  changed or what is still proposed; no side remarks about notes or memory.
- **Delivery:** small changes as an in-chat diff or a short before/after; a whole function in a
  scratch file with one copy line. Never a `.patch` file.
- **Optional items:** when a proposal mixes required and optional items, ask "all, including N?"
  so a "go ahead" is unambiguous.

## Per-function procedure

1. **Read the counterparts.** Switch: `swfunc.py FUNC` (alpha strings print first); no row is not
   "no name": check `swrun.py FUNC`, neighbours in `builds/switch/out/functions.tsv`, and the
   Switch bodies of lead-named callers (naming.md). Note what Switch inlines.
2. **Decide the function name** (naming.md): Switch name with body + position checked →
   `rename.py OLD NEW --switch` (applies when clean). No Switch symbol → a `// Name?` lead comment
   above the definition, `func_` kept.
3. **Declarations** (declarations.md): move m2c's extern lines (`tidy_externs.py` proposes a
   placement; per-TU codegen evidence overrides it), fix prototypes from callee/Switch evidence.
4. **Names inside the function** (naming.md): parameters from callee or Switch, then locals, then
   the struct fields and globals the function touches (rename every use: grep `\.field\b` and
   `->field\b` across `src/` and `include/`).
5. **Constants** (declarations.md): enums/defines for every magic number, existing ones first;
   derived values spelled from their parts; compile each.
6. **Shape** (code-shape.md): unneeded locals, initialization at the declaration, guard clauses,
   `x++`, redundant casts, case order; compile each.
7. **Comments** (comments.md): only what the code can't show; check edge values before writing a
   rule ("higher = rarer" missed that 0 disables).
8. **After applying:** full build; `make overlaps` when globals or struct fields changed; drop
   comments the new names made redundant; update the handoff's leads if a lead was settled.

Report which locals or shapes the match required (in chat, not as comments), which names are
guesses, and the rename line when a name exists. For an ambiguous name (several readings fit),
add one short reason: where the word comes from and what kind of value it is (`step`: "from the
caller's counter; the callee only tests it for 0"). The user tunes names to taste, so a solid
first candidate plus its reason beats asking (user, 2026-11-26).

## Per-function checklist

- [ ] Switch counterpart read; function renamed or `// Name?` lead added
- [ ] m2c externs moved; prototypes per evidence (local vs header)
- [ ] `argN` gone: parameters named from callee/Switch/role
- [ ] m2c temps gone or named; aliases of globals removed (compiled)
- [ ] fields/globals the function settles renamed everywhere, redundant comments dropped
- [ ] magic numbers → existing enums/defines, or new ones (compiled)
- [ ] `x -= 1` → `x--`, `!= 0` on truth tests, redundant casts/parens, init at declaration
- [ ] guard clauses where they match; no goto
- [ ] comments only for meaning the code can't show; edge values checked
- [ ] build OK; `make overlaps` if globals/fields changed

## Per-file pass

Run once the TU is fully matched. Survey first, then send one proposal listing every item with its
evidence; apply the agreed list, full build after each step, `make overlaps` at the end.

1. **Data ownership:** move the TU's `.data`/`.rodata`/`.bss` into the C file (definitions,
   string literals inline, `INCLUDE_RODATA` gone, yaml segment changes, pads the TU's own data now
   provides). String externs and their `// "contents"` comments go here. splat traps: memory
   `splat-c-file-gotchas`, `FFX/Troubleshooting.md`.
2. **Linkage:** `static` for functions and data nothing outside the TU uses (`symuse.py NAME` +
   `grep -rlE "(jal|j) +NAME\b" asm/jp_intl`, including `.word` code references); drop prototypes
   that definition order makes unneeded.
3. **Headers:** each prototype to its final home (other TUs call it → area header; TU-only → out
   of headers; `extern.h` holding entries moved out); drop unused `#include`s.
4. **One vocabulary across the TU:** unify terms and plural/singular; `const` locals repeated in
   several functions become one file define; struct definitions to the area header when other TUs
   use them, else kept in the file.
5. **Extern block:** sorted by section, then address, no duplicates (per-function cleanup leaves
   the existing order alone).

## Per-file checklist

- [ ] no `INCLUDE_ASM` left in the TU
- [ ] proposal sent and agreed
- [ ] data/rodata/bss in C, literals inline, yaml updated
- [ ] `static` where TU-only; unneeded prototypes gone
- [ ] prototypes in final headers; unused includes gone
- [ ] one vocabulary; shared consts → file defines; structs placed
- [ ] extern block sorted, no duplicates
- [ ] build OK; `make overlaps`
