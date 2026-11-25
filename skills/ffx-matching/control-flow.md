# Control-flow and layout differences

## Exits and returns (T12, T7)

- **Where the `return`s sit decides the exit layout.** Every `return` but the last jumps to gcc's
  return label; with one final `return` the label disappears and sched2 can hoist `ld ra` next to
  `v0 = ...` (movie_pause_resulti). `if (a == NULL || b == NULL) return 0;` is one return copy; two
  `if`s give two, and jump2 merges them differently (movie_status_resulti). When return paths merge
  differently than the target, vary the number and position of `return`s before anything else;
  `rtl.py --diff` shows it from the first pass.
- **Early return:** the early branch lands straight on the epilogue with v0 already set.
- **v0 not set on some path = missing or bare `return;`.** A path reaching the epilogue with no
  `move v0` (often `ld ra` in the branch slot), or a call that isn't a tail jump where `return f()`
  would make one, falls off the end and returns the last call's result (func_00163290,
  func_00162EB8, func_0015B490). Check every exit's v0 source before loop or structure rewrites;
  try deleting the final `return` early.

## Tail calls

- A `void` function whose last call is a plain statement becomes a frameless `j callee` (also to
  variadic `printf`); `return f();` keeps the frame and `jal`. Target `j f` = `void w(void) { f(); }`
  (or non-void without a return); framed = `return f();` (TkGetAsia, TksceGsSyncPath). Probe the
  exact shape before calling an instruction "non-gcc" (a value-returning probe once produced a false
  "gcc 2.9 never tail-calls").
- A void function with switch cases tail-jumping: cases end in `return;` plus `default: return;`;
  without the default the last case stays `jal`.
- A `void` `static inline` whose last statement is a call is never inlined: gcc tail-calls that call,
  and "function with sibling calls cannot be inlined" (`-Winline`). Value-returning one-call helpers
  do inline (regalloc.md getters).

## Switch

- **Case bodies are emitted in source order** (func_001625F8): sort the jump table's targets by
  address; cases sharing a target are adjacent labels (`default: case 0:`).
- **Dispatch shape** (stmt.c `balance_case_nodes`, func_00157948): 3+ cases split at the middle
  (`== mid`, then `slt`); a straight `beq` chain needs the cost table, which is off when any case value
  is a control char (1..31 except `\b\t\n\v\f`). Two nodes are never split: target
  `beqz`/`beq 1`/`beq 2`/`b` came from `if (x == 0) return;` + a two-case switch.
- **Two adjacent cases + default** = `slt hi` / `slt lo`; a range `if (x >= lo && x < hi)` gives
  `addu -lo` / `sltu` (T10); an extra case value reorders the compares (func_00164A70 disk type).
  `x > 0 && x < 7` always folds to `addiu -1`/`sltiu 6`, so two branches there = a `switch` on 1..6.
- **Switch on an enum-returning call is unsigned** (a `GameRegion` getter gave `sltiu`/`beqz`); the
  target's `slti`/`blez` needed an `s32` return or local (TkInit).
- **`while (1) { switch (step) { ... continue; } }` never threads** to direct case jumps; direct forward
  jumps between cases = fall-through + labels (func_00158A88). There, duplicated identical tails in two
  cases (cross-jumped by gcc) gave the target's extra `%hi` copy register; one shared tail via a label
  lost it.
- `x == 1` inside `case 1` can show as a compare with the switch register.

## Loops

- **Loop direction in the asm proves nothing.** gcc 2.95 reverses simple array loops: base + K,
  counter n-1 counting down with `bgez`. Source is `for (i = 0; i < n; i++)` with the array starting
  at **K - (n-1)*stride** (TkInitSaveWork: K=0x1E8, n=56 -> 0x10C; TkInit: K=0x18, n=8 -> 0x11); a
  stored counter value is `a[i] = i`. Compute the start this way before proposing struct layouts from
  a loop. Always compile the natural ascending form first, before any explanation; if forms tie, the
  natural one is the source. Missed twice: func_0015F7E0 (recommended `for (i = 7; i >= 0; i--)`
  though the variants tied) and func_002D26A0 (argued an explicit `*p--` walk; plain
  `for (i = 0; i < 4; i++) a[i] = K;` matched).
- **Test at top vs bottom is decided in the front end** (stmt.c `expand_end_loop`): a jump to the end
  label (`break`, a condition) in the first ~30 insns moves everything up to it to the bottom. A loop
  with its test at the top and body out of line has no `break`/condition near its start; its exits
  leave some other way, or a nested loop comes first (the scan stops at a nested loop's start).
  func_001632B8 matched only with `goto restart`/`goto read` after every break/continue form failed:
  a diagnostic of that rule, not evidence of the source shape (the natural form is still unknown).
- **`do { } while (1)` vs `while (1) { }`** lay out differently: jumping over a trailing sleep into
  the body on the first pass = `do ... while (1)` (func_00163D20; `while (1)` 43 words off). Retry
  loops `while (read() == 0) { do wait; while (!ready); }` match directly.
- **Function ending in an infinite loop** (func_0015B490, func_0015B3F8): jump.c moves each return's
  label behind the loop's back jump, loop.c then ignores the loop ("multiple entry points") and hoists
  nothing. Shape that works: wrap the loop (`if (c) { for (;;) {...} } return 1;`; the guard form was
  35 words off, a `do { if (x) break; ... } while (1)` only a workaround) or put it in an `else` with a
  reachable final `return`. Vary the enclosing structure (wrap vs guard), not just the loop keyword; a
  cleanup preference for guard clauses must not shrink the search. Check siblings and refs before
  saying "only X matches" (CdReadRetry in libmscd matched `while (1) { for (i = 0; i < tries; i++)
  {...} sceCdBreak(); ... }`). Once the loop is processed, allocation priority decides which hoisted
  `%hi` keeps a saved register (regalloc.md). Diagnostics there: a `goto` to a label after the loop
  exits without rotation, and `break; ... return r` after the loop was the only form tried that
  returned a compared call result without `li v0, 1`.
- **Unbounded search loop = explicit pointer walk** (func_002D22D0): a loop entered from outside gets no
  strength reduction, so a pointer stepped apart from the counter is in the source (`if (x == *p++)
  return i + 4;` in `for (i = 0;; i++)`; 180 indexed forms failed). A bounded `for`
  loop is reduced by gcc itself. Read raw `-dL` `.loop` output (rtl.py's cut dumps drop it).
- **No unrolling at -O2:** straight-line target code = straight-line source (a constant 3-pass loop
  stayed a loop, func_001596A8).
- **Multi-array loops:** the strength-reduced pointer starts at the array stored last in the body; the
  others' offsets give the source order (target offsets +0x3C, +0x28, +0 = stored in that order).
- **Backward copy:** `*dst-- = buf[n - 1 - i]` keeps an up-counting `i`; `*dst-- = *src--` is reversed.
- **`arr[q][i]` vs a row pointer:** the 2-D form strength-reduces the destination separately.
  Direct `arr[i].field` vs an element pointer: T14.
- **Conditions:** `&&` in a `for` condition breaks loop rotation (func_002D2E10: plain `for` +
  `if (...) break;` matched; the comma-operator slip there is in m2c-traps.md).

## Branches, merged and duplicated tails

- **The target puts an `if` body first**; a far jump for a condition means the source tests the
  opposite.
- **Same store in both arms = one statement after the `if`** (copied into both delay slots
  `bnel ... / sw` and `b ... / sw`; func_00163E90 `busy_index = -1`). Two identical `if (flag == 0)`
  tests that both survive need an if/else (func_00164320 `entry->size = size`; with
  `x = ...; if (f) {...} if (f) {...}` CSE merges the tests, knowing the store can't alias the flag).
- **Value pick:** temp + one store after -> `movz`. A branch per arm with its own `lui` = a store in
  each arm, tail-merged. Target `x = a; beqzl c; (slot) x = b;` + trailing store: a full if/else
  assigning both variables in each branch; the identical store cross-jumps behind the join and jump.c's
  cmov never fires (func_001AE3D0; case 0 beside `default:` and an initial value reused as a case
  constant were the other keys there). Ternary / `x == K` -> `xori`+`sltiu`.
- **The same global written on each path of an if/else** with a shared join store = stores inside
  both branches, cross-jumped (func_00159CC8).
- **Cross-jumping** (jump.c, after reload) merges only register-identical tails; a duplicated copy of
  source keeps every copy (394 vs 244 words, even 4 identical unpack blocks), but calls whose argument
  setups end the same do merge (two CdReadRetry calls, as MSVC did on PC), and so do identical
  `x = y; goto L` / `x = y; L:` tails. flow.c's
  `(use (const_int 0))` after a block-ending call blocks it. Direction (`find_cross_jump`): jumps
  are processed in insn order and the processed one is deleted into the other; a register difference
  upstream flips which copy survives (func_00158A88). There, `if (f()) break;` + one `return 0` after
  the switch let two `call; if (r) goto L_ret; tail` sequences cross-jump down to `jal; b <twin's
  test>`; `return 0` inside each case did not.
- **Duplicated identical blocks** (func_00165808): the helper can't be a `void` inline ending in a
  call (see Tail calls), and `do {} while (0)` moves blocks. The code written twice (or a bare macro) with
  function-level locals matched; per-copy locals swapped registers.
- **Delay slots show the pre-reorg layout:** a branch landing one insn past a label, with that insn
  copied into its slot = reorg stole it; a `jal` slot holding an insn from after the call = gcc's
  reorg (gas only fills backwards). Use this to place labels before writing C.
- Jump threading doesn't carry `x == 0` across an intervening call.
