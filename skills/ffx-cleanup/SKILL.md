---
name: ffx-cleanup
description: Cleaning up matched FFX (PS2) functions in ~/dev/ffx: the cleanup checklist, what to apply directly vs propose, Switch-backed function renames, local/parameter/field/global names, lead comments, constants and defines, declaration placement, and reading-quality code shape. Use when asked for "cleanup" of a function or TU, when proposing names or comments for matched code, or when reviewing the user's cleanup.
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

## Working agreement (always applies)

- **First command of every cleanup turn: `git diff --quiet; echo $?`.** Run it, don't read
  `git status`: `M ` (staged only) counts as clean. Check again right before editing, since the
  user stages WIP around their messages.
  - **0 (clean):** apply the trivial items yourself, build, then report. Trivial = a Switch-backed
    function rename (body and position checked), parameter names taken from the callee or Switch,
    moving m2c's `/* extern */` lines to their block, offset tags, enum-for-literal swaps that
    compile, agreed header/extern moves, side-by-side lead comments on settled globals.
  - **1 (unstaged changes):** propose everything; the tree is the user's.
    Also trivial when clean (evidence-backed, compiled): global renames to Switch-exported names
    whose Switch use matches ours (`rename.py --data --switch`, check the size attribute), types
    the ELF bytes or Switch settle (`char[]` for a string, plus its `// "contents"` comment),
    `// Switch Name?` leads from a
    matching call site, compiled tidy-ups (`!= 0` on flags, stray `&`, casts), prototypes into
    the existing area header (devkit.h, battle.h). "Cleanup" on a clean tree = apply all of this
    in one go, then report; don't stop for a "go ahead".
  - **Always proposed, whatever the state:** names we invent (no Switch/original source), struct
    layout changes, new defines/enums, anything uncertain or contested. "Go ahead" / "apply" on a
    proposal is permission for exactly that list; re-read the file first, the user may have
    edited it.
- **Never touch the user's in-progress function** beyond what they asked for, and never undo their
  edits or style (decimal vs hex, their comments; see comments.md).
- **Every edit is checked:** a full build (`make`, filter `grep -v 'repeat count is zero'`, expect
  `OK`) before reporting; never while the user builds (`pgrep -a make`). Comment-only edits need no
  build. Back up a file to scratch before editing it.
- **Proposals compile first.** Every code proposal (removed local, define, reordered statement)
  is tested in `variant.py SRC FUNC -e edits.py` before it is offered; say which were compiled.
  Renames and comments need no test.
- **Report as "I edited" vs "for you to apply"**, against the user's current code. List only what
  changed or what is still proposed; no side remarks about notes or memory.
- **Delivery:** small changes as an in-chat diff or a short before/after; a whole function in a
  scratch file with one copy line. Never a `.patch` file.
- **Optional items:** when a proposal mixes required and optional items, ask "all, including N?"
  so a "go ahead" is unambiguous.

## Procedure

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
guesses, and the rename line when a name exists.

## Checklist (quick pass)

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
