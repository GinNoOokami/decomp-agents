# Register allocation differences

Cheapest checks first; `vsweep pack` covers several of them in one call.

## Declarations and return types (check first)

- **Undeclared callee = implicit `int`.** gcc keeps v0 live after the call and the next load swaps
  v0/v1 (func_00158748: `void f(void);` matched, `s32 f(void);` didn't). Check every callee is
  declared; m2c's `/* extern */` block gets lost in hand tidying.
- **Callee `void` vs non-void moves registers, even far from the call.** sceIoctl and
  TkSignalSemaBGThread (func_002D1C60) swapped v0/v1 at the next load several times in
  HostReadQServe (func_00255158); its last 2 words were exactly this, missed once because a rename
  hid the old note and `vsweep pack` was skipped on the new near-match. func_002D9028's last 7 words:
  the callee func_00164A50 is `void` in its own TU (libmscd), so the original caller had no
  prototype; keep it out of shared headers and comment at the call. `vsweep pack`'s `ret:` lever
  tests this; run it early when only scattered register diffs remain.
- **Reading the prototype from codegen:** after a call declared to return a value (REG_UNUSED v0) the
  next short temp gets v1; after a `void` call it gets v0 (func_00158A88). A TU calling one function
  with 1 and 2 args had no prototype (`s32 f();`).
- **Declarations are per TU.** read_debug.c called TkSignalSemaBGThread with no prototype while
  libmscd.c and hddcopy.c saw `void`. Reproduce what each TU saw (local extern block, missing
  prototype, comment at the call) even against header tidiness. `variant.py --proto` rewrites the
  file's own declaration too, so compare with and without it (rndt: a file-local `s32
  TkGetTotalSyncVCounter()` matched; the header's `s64` adds `dsll32`/`dsra32`).
- **The function's own return type** (func_00171878): a saved-register swap (s0/s1 on two `lui`s) no
  statement or declaration order fixed was `s32` vs `void`. Check callers for v0 use first (all 8
  wrappers ignored it) and callees for real a0-a3 reads (func_001710B4 reads none: m2c's args were
  leftover stores).
- **A returned call value can show only as allocation** (func_001719E0): v0 untouched until `jr ra`,
  so `return f()` is invisible in the instructions but adds a pseudo across the tail (s2/s3 flipped
  for two saved globals; 1440 statement orders, types and flags couldn't). Check whether the callee
  leaves a value in v0 (hand-written callees too) and whether siblings return it.

## Priority and lifetimes (global.c, local-alloc.c)

- **Priority = floor_log2(refs) * refs / live_length**, refs weighted by loop depth (flow.c).
  `rtl.py --priority` shows the numbers. A missing saved register can come from another pseudo's
  choice earlier in the order. A `lui` rematerialised at its use = that pseudo lost allocation
  (reload), not a gcse difference.
- **Close priority gap = lifetime or ref-count lever, not declaration order** (func_002D2E10: 21
  orders tied). `count` (refs 4, live 37, prio 2162) beat `var_22` (refs 12, live 188, prio 2093)
  for s6 because `var_22`'s `0xFF` init at entry made its range long. Moving the inits after the first
  call (`r = scePadInfoAct(...); a = 0xFF; b = 0xFF; if (r) ...`, logic unchanged) flipped it; adding
  a ref instead (`var_22 <<= 8`) flipped it too but left an in-place `sll` and the prologue wrong. The same position decides prologue order: a CALL_INSN does
  not end a block in 2.95, so sched2 hoists those constant loads among the register saves.
- **Loop depth** (func_0015B490): a hoisted `%hi` vs a long-lived param was a near tie (0.1026 vs
  0.1031); one extra loop level around part of the body flipped it.
- **local-alloc pads lifetimes by one insn** (func_001596A8, `fake_birth`/`fake_death`): a pseudo
  born right after another dies conflicts on the first try. A swap between adjacent short temps =
  a sched1 order difference (often PRE-hoisted `%hi`s as fillers); read `.sched`, not the source.

## Variables and their shapes

- **Result variable vs direct returns** (func_001630F0): `r = 0; if (...) r = ...; return r;` makes a
  cross-block pseudo (its init is hoisted to the top, conflicting with every block-0 temp) that ends
  in a3 + `move v0`. Target computing straight into v0, constant in a branch delay slot = direct
  `return`s (`if (!empty) { if (flag) return 1; } return 0;`; reorg steals the return-0 move). Seen in
  the `.lreg`/`.greg` dumps. m2c always emits the result-variable form; a sibling using one is not
  evidence either. Same function: `!= 0`/`> 0` fold to `sra`/`andi`, a nested `if (x & FLAG)` keeps
  `lui`/`and`/`sltu`.
- **Two faults can mask each other:** that case took ~30 variants because the return shape and the
  flag test were each tried only with the other wrong. When two levers each "fail", try them together.
- **Repeated getter calls = separate variables** (TkInit func_00125E40): each `jal
  TkGetFFXCofingAdrs` is real (external calls never merge). One reassigned `config` local was 29
  words off (one pseudo, one hard register across all ranges); three locals, block-scoped locals or
  one-call `static inline` helpers match. The user prefers the helpers (no point calling a getter twice
  in one function). Plain `static` isn't inlined at -O2 (it would leave a standalone copy).
- **Reusing one variable** for two loops or two strings changes allocation (func_00254AD0: the
  `$USER$` copy used `k`). A stray copy (`daddu a2, a1, zero`) used only for a loop's first test = a
  variable assigned twice (one pointer reused for two strings), so CSE can't merge it; jump.c's
  `duplicate_loop_exit_test` gives the copied test fresh pseudos (found by the permuter in ~14k
  iterations).
- **Test the field, don't copy it first:** `if (desc.lsn == 0) ... else lsn = desc.lsn;` vs
  `lsn = desc.lsn; if (lsn == 0)` reallocates the whole function. A plain copy (`len = rest;`) is
  coalesced away.
- **Unused initialised locals still count** (MsBtlListFieldNum: a leftover `field = &list[i]` cost 4
  words). After a refactor, delete old locals before judging a form.
- **Uninitialised m2c temps cost a stack slot** (bigger frame, `lw a1, 0(sp)`).
- **Frame bigger by a spill slot + an extra `lui` hoisted at entry** = one more long-lived value in the
  source (func_00164628: a string pointer local set right before the `if`, not at its declaration or
  inside; a constant local, so flag it, but the PC sibling keeps a name pointer `_Dst = "DVD FILE"`).
- **Address-taken locals** are reloaded after every call, so a `tmp = fid + 1` reused across calls is
  a real source temp (inlining it: +169 words).
- **Declaration order sets spill-slot order** only (func_002D9028: table 0x140, hddEnabled 0x144,
  path 0x148 needed those three declared first); register-only locals' order does nothing (720
  permutations, func_00163450).
- **Narrow types add extensions:** an `s8` counter loaded with `lb` and decremented must be `s32`
  (174 -> 98 diff); byte-typed params kept across a call add one too (TkSetEventNewGame).
