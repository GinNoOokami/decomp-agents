# m2c output that misleads

Troubleshooting covers the recurring ones: inlined copies (T4), a struct of function pointers called
as a global (T5), undeclared `sp` locals (T6), guards hiding a return value (T7), signed division
(T8), wrong call arguments (T9), range checks (T10), `%hi` loaded on two paths (T17), `!= K` counters
(T19). Extra points:

- **Arguments set in a tail call's delay slot are dropped** (`j func` + `addiu a2, zero, 0x14D`, the
  assert line passed to func_00254F18); m2c shows them missing or invents params. Read the slot.
- **Argument registers set before a `jal` are real arguments** even if the callee ignores them
  (`a1 = name` into func_00254F18): the callee has an unused parameter.
- **Callee parameter guesses come from leftover registers.** Check the callee's first use of a0-a3; a
  delay-slot `nop` where an argument load would go = no argument (func_00166878: none,
  func_00157788: three). m2c also invents args from `%hi` registers live at a `jal` (TkInitScene's
  `func_001BE3B8(&D_..)`) and drops pass-through a0 args (TkMsImportantClear).
- **A comma operator in a `for` condition** (an m2c-translation slip) silently drops the left test.
- **The jump register becomes an argument** (`lw $4; jr $4`): unprototyped callees then get
  `f(temp_4)` in every case; prototype them `void (void)`.
- **Every `temp_*` argument must be assigned**; leftovers cost a stack slot. Text-anchored edits can
  hit m2c's commented copy first.
- **`var = 0; if ...; return var;` is m2c's habit**, not evidence of a result variable (regalloc.md).
- **A guard's return value** hidden by nesting: `decompile.py FUNC --print --gotos-only`.
