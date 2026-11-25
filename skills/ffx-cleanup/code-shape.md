# Code shape (reading quality)

This is the overlap with matching: the code must read like what a programmer typed. Whether a
shape compiles the same is a matching question (`ffx-matching`: control-flow.md, types-and-shapes.md);
here every change is compiled with `variant.py -e` before it's offered, and the matched shape wins.

## Locals

- **Cut what adds nothing:** m2c temps, single-use copies, aliases of globals
  (`Unk* act = g_CurrentAction;`), a copy of a field read twice before any store
  (MsSetBattleEncount's `weight`). One variant per removal, plus one with all; keep a local only if
  the match needs it, and say so in chat.
- **Readability wins over fewer locals:** a named intermediate that splits a compound expression
  stays, even if removing it matches (`s32 size` in `TaskBuffInit(n, f(TaskBuffSize(n)), 1)`;
  `fieldId` beside `entryId` documents how a packed id splits). A local that only exists to shorten
  a line clang-format would wrap is fine when it matches.
- **Initialize at the declaration** (`u8 oldWeight = 0;`), not m2c's split `u8 x;` + `x = 0;`,
  whenever the match allows.
- **Scope:** block-scoped locals where the value lives in a block are fine
  (`s32 roll = brnd(1) % …` inside the hit branch).

## Statements and expressions

- `x -= 1` / `x += 1` → `x--` / `x++`; drop `!= 0` on plain truth tests (keep `!= NULL` if the file
  uses it); drop redundant casts and parentheses (`(s16)fieldIndex` into an `s16` field).
- Literal bases as the file has them (`>> 16` beside `>> 16`, not `>> 0x10` in one place).
- **m2c's comma conditions** (`if (a && (s1, s2, test))`) are statements before nested tests;
  m2c's `var = 1` before a call is often argument setup for the call, not a statement.
- **Wrappers:** a bare `j callee` target is `void w(...) { f(...); }`; `return f();` keeps a frame.
  Return a value only when callers use v0.

## Control flow

- **Guard clauses and early returns** where they match (MsBattleEncountExe: seven `return 0`
  guards, each exit setting v0 in its branch's delay slot); merge nested `if`s with `&&` when it
  matches (func_001AE858).
- **No goto, no empty `do {} while (0)`.** If only such a shape matches, the natural one hasn't been
  found: that's an `ffx-matching` question, not a cleanup result.
- **`for` loops for counted walks;** a `while` with the test first when the target jumps straight
  to the test.
- **Switch cases in natural order** (ascending, or enum order), inner switches too; case order
  moves code, so check with a full build.

## Not cleanup

- Layout (one-line bodies, braces, wrapping, literal case) belongs to `.clang-format`; never flag it.
- Style nits outside the function, and items for the project-wide `FFX/Cleanup.md` (that file takes
  only faithfulness items across the codebase; per-system items go in the system's notes, small
  tidy-ups only in chat).
