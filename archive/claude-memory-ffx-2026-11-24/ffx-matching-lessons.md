---
name: ffx-matching-lessons
description: "Verified ee-gcc 2.9 / m2c patterns from FFX matches (tail-call delay-slot args (Troubleshooting 9b), signed-division rounding, void vs value return, operand order); check against target asm before hinting"
metadata:
  node_type: memory
  type: feedback
  originSessionId: d8b8ba71-d5b2-44ee-a8a4-83ea356c0bf1
  modified: 2026-11-05T18:47:30.987Z
---

Verified in scratch on FFX (ee-gcc 2.9-991111 `-O2 -G0`), 2026-10-30, read_debug.c. Use to check hints,
never to hand the user code unasked ([[no-unrequested-decomp-answers]]).

- **m2c drops arguments set in a tail call's delay slot** (`j func` + `addiu a2, zero, 0x14D`): it shows
  them as missing or invents params. Read the delay slot (func_00254F18's assert line 333).
- **Extra argument registers set before a `jal` are real arguments** even if the callee ignores them
  (`a1 = name` into func_00254F18): the source passes them; the callee gets an unused parameter.
- **`(x > -1 ? x : x + (D-1)) >> k` then `<< k`** (m2c `movn` form) = signed `(v + D-1) / D * D`; writing
  `& ~(D-1)` gives an `and` instead.
- **`void` return lets gcc turn a final call into `j` with no frame**; target keeping `jal` + frame + `v0`
  intact means the function returns the call's value.
- **Return type also moves register allocation** (2026-11-05, user's func_00171878 match): a
  saved-register swap (s0/s1 on two `lui`s) that no statement/decl order fixed was `s32` vs `void`
  return. Before brute force, check callers for v0 use (all 8 wrappers ignored it) and callees for
  real argument reads (func_001710B4 reads no a0-a3: m2c's args were leftover stores).
- **Switch dispatch shape** (stmt.c `balance_case_nodes`, 2026-11-06, func_00157948): 3+ single cases split at the middle (`== mid`, then `slt` tests); a straight `beq` chain needs the cost table, which is off when any case value is a control char (1..31 except \b\t\n\v\f\0). Target `beqz`/`beq 1`/`beq 2`/`b` came from `if (x == 0) return;` + a two-case switch (2 nodes are never split). Also: `x == 1` inside `case 1` can show as a compare with the switch register (m2c prints the global).
- **`switch` case bodies are emitted in source order** (2026-11-01, user's func_001625F8 match): the
  jump table is indexed by value, so sort its targets by address to get the source order of the cases;
  cases sharing a target are adjacent labels (`default: case 0:`).
- **m2c passes the jump register as an argument**: gcc may load the table entry into a0 (`lw $4; jr $4`);
  with unprototyped callees m2c then writes `f(temp_4)` in every case. Prototype them `void (void)`.
- **`lw` from a global before indexing** = the global is a pointer (`s16* p; p[i]`), not an array.
- **`addu` operand order can depend on the source**: `index = state->unk20 + id` matched,
  `id += state->unk20` / `id + state->unk20` did not (func_00254E08).
- **Store shapes and types read from the asm (2026-11-07, all verified by a match):** (1) a store
  through a call's result into a *struct field* (`((TaskSlot*)create(...))->args[0] = G;`) lets gcc
  keep global G in a register afterwards; a plain `*(T**)p = G` store forces a reload of G
  (func_00159CC8; gcc 2.95 treats struct-member stores as not aliasing plain globals, inferred).
  (2) A float constant stored via an integer register (`lui 0x3F80; sw`) = an integer field holding
  the bits; a `float` field goes through `mtc1/swc1` and shifts the saved-register count
  (func_001581E0). (3) `lbu` + `dsrl` of a flag byte = a 1-bit `u32` bitfield, narrowed by gcc;
  `(u64)*p >> 7` and u8/u64 bitfields don't give it (func_00158FC8). (4) gcc reassociates
  `x + C - w / 2`; separate `x += C; x -= w / 2;` keeps the target's order (func_001581E0).
  (5) The same global written twice on each path of an if/else (with a shared join store) = all
  stores inside both branches, cross-jumped (func_00159CC8).
  (6) Target `x = a; beqzl c; (slot) x = b;` + a trailing store, where ours gives `movz`: write a
  full if/else assigning both variables in each branch; the identical store cross-jumps behind the
  join and jump.c's cmov never fires (func_001AE3D0, 2026-11-23). Case 0 beside `default:` + an
  initial value reused as a case constant (`beq s0,s2`) were the other keys there.
- **Where the `return`s sit decides the exit layout** (2026-11-07, verified with `rtl.py --diff`). (1) Every `return` but the last jumps to gcc's return label; with one `return` at the end the label disappears, the epilogue's `ld ra` shares a block with `v0 = …` and sched2 hoists it (movie_pause_resulti: target `daddu v0,zero,zero` before `ld ra` = some non-final `return`; per-branch returns and a guard clause both match). (2) `if (a == NULL || b == NULL) return 0;` expands to ONE `return 0` copy; two separate `if`s give two, and jump2's cross-jumping then merges the copies differently (movie_status_resulti case 2: guards sharing one `jr ra; daddu v0,zero,zero` while the default's `v0 = 0` branches to a final `jr ra` = split guard). Visible from the first pass (`rtl`), so `--diff` shows it at once. When return paths are merged differently than in the target, vary the number and position of `return` statements before anything else.
- **Single exit vs early return** shows in where the early branch lands: straight to the epilogue with
  `v0` already set = early `return x;`.
- **Scheduler reverses global-address order**: stores to several globals came out with `lui`s in reverse
  of source order; source in call order matched (func_002556F8).
- **Callee return type shifts regalloc after the call**: `void` callee gave `lui v0`/`lb v1` for a later
  global; declaring it `s32` (it does set v0) flipped to the target's `lui v1`/`lb v0`.
- **`relocation truncated to fit: R_MIPS_GPREL16`** at link = a C function's size is wrong, shifting
  `.data` past `$gp` reach (D_0057C200 sits 0x7FF0 below gp 0x5841F0, so ~0x10 bytes of shrink breaks it).
- **A stray register copy (`daddu a2, a1, zero`) used only for a loop's first test** = a variable
  assigned twice (one pointer reused for two strings), so CSE can't merge it with the value it
  was copied from. jump.c's `duplicate_loop_exit_test` gives the copied exit test fresh pseudos;
  they survive only then (func_00254AD0, found by decomp-permuter in ~14k iterations).
- **Reusing one index variable for two loops** changes which registers get allocated
  (func_00254AD0: the `$USER$` copy used `k`).
- decomp-permuter: never set up by hand; `~/dev/ffx-analysis/scripts/perm.py go <func> [--base f.c]`
  then `perm.py watch/status/best/stop` (2026-11-04). It handles context, inline bodies, the
  fd_set re-print, target.o (wrapped in `.set noat`/`.set noreorder` like INCLUDE_ASM; without it gas
  refills delay slots: all hand-built permuter runs before 2026-11-04 scored against a mangled target)
  and checks the re-print compiles. Differing relocation symbol names cost points (no 0 with a literal
  where the .s has a D_ label). Read every result: the permuter's
  improvements are often semantic changes or noise (e.g. a `(float)` cast).
- **Callee return type moves regalloc**: declaring a callee non-void (sceIoctl, func_002D1C60 =
  TkSignalSemaBGThread) swapped v0/v1 at the next load several times in func_00255158 (= HostReadQServe)
  — check before other levers. Missed again 2026-11-17 on HostReadQServe: the rename hid this entry
  from my search and I skipped `vsweep pack` (its `ret:` lever) on the new near-match; the last 2
  words were exactly this (no prototype in scope = implicit `int`). Old func_/D_ names in memory may
  be renamed: grep symbol_addrs for the address too.
- **`extern volatile`** for a flag polled in wait loops fixed reload/hoisting (func_00255158).
- **Branch layout**: the target puts an `if` body first; jumping far for a condition means the
  source tests the opposite (`if (D_00565DF4 != 0)` first).
- **gcc 2.95 internals behind shared tails** (from the PS2 Linux 2.95.2 source, RTL dumps `-da`):
  cross-jumping (jump.c, after reload) only merges identical tails; flow.c puts
  `(use (const_int 0))` after a call that ends a block before a label, which blocks it. sched1
  (sched.c) won't hoist an insn above a call only if its dest crosses no call
  (`REG_N_CALLS_CROSSED == 0`); otherwise it may. Jump threading doesn't carry `x == 0` across an
  intervening call. EE gcc tail-calls a void function's last call, so a `static inline` helper
  ending in a call won't inline ("function with sibling calls cannot be inlined").
- **Probe passes directly**: compile with `-dS`/`-dR` in a scratch dir and check where an insn
  sits after sched1; faster than scoring whole variants.

Related: [[verify-hints-before-giving]], [[logic-first-then-match]].
- **Loop test at top vs bottom is decided in the front end** (`stmt.c` `expand_end_loop`, visible already in the `.rtl` dump): if a loop's first ~30 insns contain a jump to its end label (`break`, a `while`/`for` condition), everything up to that jump is moved to the bottom (`b test; body; test`). The scan stops at a nested loop's LOOP_BEG. So a target loop with its test at the top and body out of line cannot contain `break`/a condition near its start: its exits are `goto`/`return`, or a nested loop comes first. `jump.c`'s `duplicate_loop_exit_test` and the "if (foo) bar; else break" swap were not the cause (func_001632B8, 2026-10-31: `goto restart`/`goto read` matched 100% after every break/continue form failed).
- **Field types steer scheduling through alias sets**: RTL `(mem/s:SI ... N)` carries the type's alias set; stores of different types (e.g. `void*` vs `s32`) are known not to conflict, which drops their dependence chain and changes sched1 priorities. func_00162DE0: an `unk4` copy sat one insn early with any pointer type; `u32` matched but is a fake (every use is a pointer). Rules (c-common.c `c_get_alias_set`): signed/unsigned ints share a set, each pointer type has its own, union-member and bit-field accesses get set 0 (alias everything). A union field (`union { void* p; } unk4`) matched with a pointer type; still open whether the original had that, an outer union, or something else. Check `-dS` ready lists / alias numbers before trying statement orders.
- **Direct `arr[i].field` vs a pointer local**: indexing a global folds a word field's offset into the base (`(sym+8) + i*size`), giving each field its own address that later becomes a register copy; byte fields and offset 0 share `sym + i*size`. Several copies of one address = direct indexing in the source. Different symbols can't be CSE-related, so a struct member reached via another field's address (`lw -0x80(a1)`) means one symbol; separate `lui`s mean separate globals.
- **Cast around a call result moves the store's address before the call** (expr.c:3436 `expand_assignment`: a bare CALL_EXPR RHS is evaluated first; a cast is not a bare call). Sign: the global's `%hi` sits in a saved reg (`s0`) after the `jal` (sched2 moved the lui down). func_00162CC8: `D_00595AEC = (Unk00595AEC*)func_0011D370(...)`. An implicit conversion counts too: func_00159668 matched as plain `D_0030D07C = func_00173A98(); if (D_0030D07C & 0x01000000) ...` once the getter returned `u32` into the `s32` global (`s32` getter: 11 words off; a one-use inline helper had matched by accident, 2026-11-07). Check callee return vs destination type before odd rewrites; `rtl.py --diff` shows it as `%hi` before the call in the cse pass. A chained `G = tmp = call()` that "needs" the temp is the same cause: with `void*` call and `void*` global there is no conversion, so the plain store evaluates the call first; func_00159EA8 matched plain once `D_0058F9DC` was `u8*` (any non-`void*` type works; full build OK, 2026-11-07, Troubleshooting 11). Read the type, don't keep the temp.
- **`lui 0xffff; ori 0xffc0` align mask = unsigned operand**: `ALIGN_64((u32)p)` matches; `ALIGN_64((s32)p)` gives `addiu -64` (func_00159EA8, 77 words off, Troubleshooting 13).
- **Exit leaves v0 unset = bare `return;` in an `s32` function** (func_0015B490, 2026-11-08; rule was only in the FF7 project memory until now): if one exit returns whatever is already in v0 (a call's result, e.g. `sceCdRead(...) == 1` then straight to the epilogue with no `li v0, 1`) while other exits load a constant, the source had `return;` (C89 allows it; gcc only warns). Check every exit's v0 source before trying loop/structure rewrites. Also check this when our own comment says "returns X's v0 without reloading".
- **Check Troubleshooting.md's index before hinting** (grouped by symptom, 2026-11-06): inlined copies (4a memcpy / 4b unused by-value struct param), nesting that hides a guard's return value (7), signed division/modulo expansions (8), pass-through arguments m2c drops (9), unsigned range checks (10).
- **An undeclared callee changes codegen** (func_00158748, 2026-11-06): with no prototype gcc assumes `int` return and keeps v0 live after the call, swapping v0/v1 in the next load. `void f(void);` matched, `s32 f(void);` didn't. Check every callee is declared (m2c's `/* extern */` block lost when tidying by hand) before chasing register swaps right after a call.
- **Don't trust m2c's callee parameter guesses**: it reads leftover argument registers. Check the callee's first use of a0–a3 (func_00166878 overwrote a0 → no args; func_00157788 actually took 3). A delay-slot `nop` where an argument load would go = no argument.
- **Result variable vs direct returns** (func_001630F0): `r = 0; if (...) r = ...; return r;` makes r a
  cross-block pseudo (its init is set in block 0, sched1 hoists it to the top), so it conflicts with
  every block-0 temp; local-alloc gives those temps v0 first and global-alloc puts r elsewhere (a3)
  + `move v0` at the end. Target computing straight into v0 with `daddu v0,0` in a branch delay slot =
  direct `return`s (`if (!empty) { if (flag) return 1; } return 0;`): reorg steals the return-0 move.
  Seen in the -da dumps (.lreg live-at-end, .greg conflicts); `!= 0`/`>0` fold to sra/andi, a nested
  `if (x & FLAG)` keeps lui/and/sltu.
  Why it took ~30 variants: two independent faults masked each other (return shape + flag test);
  each fix was tried only with the other still wrong, so each looked like a dead end. When two
  levers each "fail", try them together. m2c always emits `var = 0; if…; return var;` and a matched
  sibling used one too: neither is evidence the original had a result variable. Signal for direct
  returns: result computed straight into v0, constant in a branch delay slot.
- **Duplicated identical blocks** (func_00165808, 2026-10-31): ee-gcc won't inline a `static inline`
  ending in a tail call ("function with sibling calls cannot be inlined", see `-Winline`), so the
  helper stays a real `jal` function. A `do { } while (0)` macro wrapper changed block layout (moved
  the then-block to the function end). What matched: the code written twice (or a bare macro) using
  function-level locals shared by both copies; per-copy locals swapped two registers.
- **Scratch compare harness: keep branch targets** (2026-11-01, func_00165B18): masking branch
  offsets to `L` hid a `break` vs `return` difference (bltz to the reset store vs the epilogue) and
  I told the user they were equivalent. Single-function objects start at offset 0 in both my .o and
  the assembled target .o, so compare real offsets; mask only relocated immediates.
- **Scratch checks: compare the assembled object, never the `-S` text** (2026-11-01). ee-gcc emits
  most branches under `.set reorder`; reorg only fills some slots (`.set noreorder` blocks). In
  reorder blocks the line after a branch is just the next insn and gas inserts the `nop`. Reading
  `-S` as if that line were the delay slot cost a wrong "unsolved difference" on func_001648F8.
  Use `~/dev/ffx-analysis/scripts/variant.py` (compiles the real file, diffs against the load image;
  `-e`/`-f` for variants) instead of ad-hoc compile loops.
  variant.py false MATCH (fixed 2026-11-04): before that date it compiled without -DNON_MATCHING, so on a tree
  file a nonmatching function's INCLUDE_ASM was diffed against itself. Distrust earlier tree-file results.
  variant.py traps (2026-11-02): `-e` edits replace the FIRST occurrence in the whole file, and
  `s32 i;` or `s32 func_X(` also match earlier functions or the top prototype, so anchor on text
  unique to the function. `-f` needs an existing C definition: on a scratch copy, swap the
  `INCLUDE_ASM` for a stub. If a header prototype conflicts with a scratch signature, wrap the
  include in `#define func_X func_X_hdr` / `#undef func_X`.
- **Missing `return` is a real FFX pattern; suggest it early** (seen twice by 2026-11-01:
  `func_00163290`, `func_00162EB8` in libmscd.c). Signs in the target: a path reaches the epilogue
  with no `move v0, …`, i.e. a branch jumps past the shared return-value copy (often with `ld ra`
  in its slot), or a call isn't a tail jump where `return f()` / `void` would make one. That path
  falls off the end and returns whatever v0 holds (usually the last call's result). Adding a
  return changes register allocation or the tail-call shape. Before trying many variants for a
  "v0 already set" layout, try deleting the final `return`.
- **Renames: also compile with `-DNON_MATCHING`** (2026-11-01). `#ifndef NON_MATCHING` blocks
  hold C that the normal build never compiles, so a compiler-guided rename misses them (103 field
  accesses in func_00163950 / func_00255158 the first time). Check both builds after any rename.
- **Batch from 2026-11-02 (libmscd func_00163450/165588/164628/163768), all verified:**
  - **Brute-force statement order with variant.py `-e`** before reasoning about regalloc: all 120
    orders of the global stores (func_00163450: only lsn, size, buff first matched) and every
    single-statement move of an 11-store block (110 variants, one match, func_00165588). Local
    declaration order changed nothing (720 permutations). Store-order hint: the value loaded last
    goes in `v1` and is stored first.
  - **Constant-size `memcpy` is inlined**: `(src|dst) & 7` test, then 32-byte `ld/sd` and
    `ldl/ldr`+`sdl/sdr` loops. m2c shows `unknown instruction: ldl` and, from the leftover `$9-$11`
    (EE a5-a7), invents extra s64 callee arguments. Write `memcpy(dst, src, CONST)`.
  - **void function, both switch cases tail-jumping**: cases ending in `return;` plus
    `default: return;`. Without the default the last case stays `jal`; an s32 `return f()` was worse.
  - **Frame bigger by a spill slot (`sw a0, 0(sp)`) and an extra `lui` hoisted at entry** = one more
    long-lived value in the source. func_00164628: a string pointer local set right before the `if`
    (not at its declaration, not inside). Constant local, so flag it; the PC port's sibling keeps a
    name pointer (`_Dst = "DVD FILE"`), so it may be real.
  - **2-D global index `arr[q][i]` vs a row pointer `line[i]`**: the 2-D form strength-reduces the
    destination into its own incrementing register; the row pointer indexes both sides.
  - **Backward byte copy**: `*dst-- = buf[n - 1 - i]` keeps an up-counting `i` (`slt i, n`);
    `*dst-- = *src--` gets reversed into a down-counter.
  - **Assert strings name locals**: `"q_cnt != READ_Q_CNT"` gave the loop index `q_cnt`.
- **Batch from 2026-11-02 (libmscd func_00164040/164320/163E90/163D20/164A70), all verified:**
  - **Same store in both arms → a shared tail after the `if`**: a store copied into both delay
    slots (`bnel … / sw` and `b … / sw`) is one statement after the if (163E90 `busy_index = -1`,
    164320 `entry->size = size` in an if/else when two identical `if (flag == 0)` must both survive:
    with `x = …; if (f) {…} if (f) {…}` CSE merges the tests because it knows the store can't alias
    the flag).
  - **`do { … } while (1)` vs `while (1) { … }` lay out differently**: the target jumping over a
    trailing sleep into the loop body on the first pass = `do … while (1)` (163D20; `while (1)` 43
    words off). Retry loops `while (read() == 0) { do wait; while (!ready); }` match directly.
  - **Two adjacent `case`s + default = `slt hi` / `slt lo`**; a range `if (x >= lo && x < hi)` gives
    `addu -lo` / `sltu`. An extra case value reorders the compares (164A70 disk type).
  - **`lui 0xffff` / `ori 0xf800` = an unsigned mask** (`0xFFFFF800`, `~0x7FFU`, or `~0x7FF` on a
    `u32`); `li -2048` = `~0x7FF` on a signed value.
  - **`lb` on a byte field = `s8`**, `lbu` = `u8`: fix the header type (ReadQEntry +0x30).
  - **`%lo(sym + 0x4)` stores with no pointer load = a separate global** splat had no symbol for
    (`D_0057C700 + 0x4` was not `D_0057C700->field`); add it to symbol_addrs.
  - **Test the field, don't copy it first**: `if (desc.lsn == 0) {…} else { lsn = desc.lsn; }` vs
    `lsn = desc.lsn; if (lsn == 0)` changes allocation of the whole function.
  - **Store order: brute-force permutations, prefer the one equal to the PC port's order** when
    several match (163D20, 164A70 end stores).
  - **Uninitialised m2c leftovers cost a stack slot** (bigger frame, `lw a1, 0(sp)`): check every
    `temp_*` argument is assigned; edits anchored on text can hit m2c's commented copy first.
- **Batch from 2026-11-04 (ReadQEntryRead/HostReadQServe tools pass):**
  - **Delay slots reveal the pre-reorg layout**: a branch landing one insn past a label, with that
    label's first insn copied into its slot = reorg stole it (the RTL jumped to the label). A `jal`
    slot holding an insn from *after* the call = gcc's reorg filled it (forward); gas only fills
    backward (`.set reorder`). Use these to place labels before writing C.
  - **Copied source doesn't merge**: cross-jumping (jump2, after reload) needs register-identical
    tails; a fully duplicated version kept every copy (394 vs 244 words), even 4 identical unpack
    blocks. It *does* merge calls whose argument setups end the same (two CdReadRetry calls merged,
    as MSVC did on PC), and identical `x = y; goto L` / `x = y; L:` tails.
  - **Missing saved register = allocation-order chain**: global.c allocates call-crossing pseudos by
    floor_log2(refs)*refs/live_length; the culprit can be another pseudo's choice earlier in the
    order (`rtl.py --priority`). A target `lui` rematerialised at its use = that pseudo lost
    allocation (reload), not a gcse difference. A plain copy in C (`len = rest;`) gets coalesced.
  - **ee-gcc 2.9**: no `-iquote` (the dir becomes a second input file: "cannot specify -o with
    multiple compilations"); its cpp prints a bare `#define` for every macro with `-dD`.

**ee-gcc 991111-01 -O2 probes (2026-11-04):** a `void` function ending in a call is a frameless tail
`j callee` (also to variadic `printf`); a value-returning `return f(x);` keeps the frame + `jal`.
gcc's own leading nops: `nop; nop` before `sqrt.s` (FPU hazard), `nop` x5 + `b` for `for(;;);`
(short-loop pad). I once claimed "gcc 2.9 never tail-calls" from a returning-value probe: probe the
exact shape (void vs value) before calling an instruction non-gcc.
- **A returned call value can show only as register allocation** (2026-11-04, func_001719E0): when
  `v0` from the last call is untouched until `jr ra` (stores and restores after the call don't use
  v0), `return f(...)` via a local is invisible in the instructions but adds a pseudo live across
  the tail, which flipped s2/s3 for two saved globals. 1440 statement orders, types, flags couldn't.
  Check: does the callee leave a value in v0 (hand-written too), and do siblings return the call?
- **`x -= t - C` is reassociated by fold()** (2026-11-05, func_00171AB0; fold-const.c `associate:` +
  `split_tree`): becomes `(x + C') - t`, then CSE folds `x + C'` into a known base (`task + 56`).
  Target `addiu t,t,-C; subu x,x,t` needs `t -= C; x -= t;` (or a separate variable): fold only
  splits a direct PLUS/MINUS subexpression.
- **Load/store order under -fno-strict-aliasing pins statement order**: a load from a global table
  can't move above a store through a struct pointer, so a table load that precedes a field store
  in the target precedes it in the source. Use it to prune order searches.
- **Shared rodata literal**: when an asm sibling still uses a string symbol kept by INCLUDE_RODATA
  (`D_0056BD90` "sa_task.c"), the C must pass that extern, not a literal (+16 bytes otherwise).
- **Return label pulled into an infinite loop → loop.c skips it** (func_0015B490, 2026-11-05, seen in -dL dumps): jump.c (~line 418, "unconditional jump preceded by a USE") moves each `return x;`'s USE + a new return label to just after the BARRIER before the label. When the function ends in `while (1)`, that barrier is the loop's back jump, so the label lands before NOTE_INSN_LOOP_END and the dump says "Loop at N ignored due to multiple entry points": nothing is hoisted. Fix shape: put the loop in an `else` and end with a reachable `return result;` (the other returns then find an identical USE and leave the label outside). Then reg-alloc priority (refs*log2(refs)/live length, refs weighted by loop depth) decides which hoisted `%hi` keeps a saved reg vs spilling params.
- **Loop depth weights reg-alloc refs** (func_0015B490, 2026-11-05, .lreg/.greg dumps): flow.c adds loop_depth per use to REG_N_REFS, and global.c priority = refs*floor_log2(refs)/live_length. A hoisted `%hi` vs a long-lived param can be a near tie (0.1026 vs 0.1031); one extra loop level around part of the body flips it. When the target spills params that yours keeps in s-regs, compare these numbers before rewriting. A `goto` to a label after the loop (not the loop's end label) exits without the front end rotating the loop, and `break; ... return r` after the loop is the only form that returns a compared call result without `li v0,1`.
- **Which loop constant gets hoisted = statement order** (func_002D1EA0, 2026-11-10, `.loop`/`.lreg` dumps; loop.c ~1835 `move_movables`): movables are taken in insn order while `threshold*savings*lifetime >= insn_count`, threshold -3 per move, so only the first N constants leave the loop. Of the hoisted ones, the last moved has the shortest live range and wins the last saved reg; the rest are rematerialized in the loop. Target keeps an earlier constant (0x10000) in `s8` and a later one (0x1000) in the loop = something else took the last move slot first: storing `entry` (a `%hi/%lo` address) before `stackSize` did it. Read the "moved to / not desirable" lines, then permute the stores in that one group (720 orders took ~3 min). Repeated `arr[i].f` with a 0x30 struct also kept the `*48` constant from moving, so no advancing pointer; a `ThreadParam*` local was required (81 vs 72 insns in every matching order).
- **Check scratch inputs compile cleanly before analysing their RTL** (func_002D1EA0): a "makes pointer from integer" warning meant `D_00789720[i]` on a 1-D `u8` array was a byte load, so a whole RTL comparison was built on wrong code. Fix what the target plainly shows first (advancing pointer = indexed by `i`, every store present) before threshold/order work.
- **Guard clause vs wrapping `if` around a final infinite loop** (func_0015B3F8/B348, 2026-11-05; shape from the khasinski ref repo, verified here): `if (c) { for (;;) { ... return 0; ... } } return 1;` matches; the guard form `if (!c) return 1; for (;;) {...}` is 35 words off, and `do { if (x) break; ... } while (1); return 0;` was only a workaround. Cause: jump.c's return-label move (entry above) needs a reachable statement after the loop. When testing loop shapes, vary the enclosing structure (wrap vs guard, else vs early return), not just the loop keyword; my cleanup preference for guard clauses must not leak into the search space. Check siblings (CdReadRetry in libmscd = matched `while (1) { for (i = 0; i < tries; i++) {...} sceCdBreak(); ... }`) and ref repos before saying "only X matches".
- **Struct-field reads through a pointer get hoisted above global stores** (movie_start_debug_resulti, 2026-11-08, `.sched` dump; gcc 2.95 alias.c `fixed_scalar_and_varying_struct_p`, applies under -fno-strict-aliasing): a `mem/s` (MEM_IN_STRUCT) load via a varying address is assumed not to alias a store to a fixed scalar global. `p->f` and `p[1]` ("pointer plus offset") count as struct; plain `*p` doesn't. Target loads that stay below `movie_buff_org`-style stores = plain derefs. The target's second register holding a copy of the arg (`daddu s3,s1` at entry) came from a cursor copy that gets stepped: `s32* args = arg1; ... flags = *args++; ptr = (u8*)*args;` (a never-modified copy is folded back by gcse cprop). `(u8*)*++p` also matched but reads as a hack; offer the readable form. The user isn't convinced the cursor is the original shape (matches, shape unconfirmed); they stopped there because it's debug code. Treat it as one way to get the behaviour, not as known source style.
- **`while (1) { switch (step) { ... continue; } }` never threads to direct case jumps** (func_00158A88, 2026-11-08): each `continue` goes back through the table jump, global or local step. A target with direct forward jumps between cases = fall-through + labels. Duplicated identical tails in two cases (cross-jumped by gcc) are what gave the target's extra `%hi` copy register; one shared tail via a label lost it.
- **Unused call result steers the next temp off v0** (func_00158A88, 2026-11-08, verified by variants): after a call declared to return a value (REG_UNUSED v0), the next short-lived temp gets v1; after a void call it gets v0. So `addiu v1,…; sb v1` right after a call = callee returns a value; `lw v0,…` right after = declared void *at that TU*. Reads the prototype from codegen; a TU calling one function with 1 and 2 args = no prototype (`s32 f();`).
- **jump2 cross-jump direction** (jump.c `find_cross_jump`, same function): jumps are processed in insn order and the one being processed is deleted into the other copy; a label at the start of stream 1 lowers the minimum by 1, so a lone `v0=0; jump exit` after a label merges into another one. A tail that already merged into a neighbour (same reg for the constant) can't merge with its twin, so a register difference upstream flips which copy survives. `if (f()) break;` + one `return 0` after the switch lets two `call; if (r) goto L_ret; tail` sequences cross-jump down to `jal; b <twin's test>`; `return 0` inside each case does not.
- **local-alloc pads lifetimes by one insn** (func_001596A8, 2026-11-08, local-alloc.c ~1420 `fake_birth`/`fake_death`, active because sched2 runs): a pseudo born on the insn right after another dies counts as conflicting on the first try, so two back-to-back temps (e.g. fn-pointer loads around a call) get different regs. A one-register swap between adjacent short temps = a sched1 order difference (what sits between the death and the birth), often PRE-hoisted `%hi`s (gcse) used as fillers; check `.sched` order, not the source statement.
- **Loop direction in asm proves nothing** (func_0015F7E0, 2026-11-08; user: "You should always validate whether a loop is actually written reverse, or if it's just the compiler doing its usual tricks"): gcc 2.95's loop optimizer counts down to 0 (`bgez`) and may walk a pointer either way. Always compile both the natural `for (i = 0; i < n; i++)` and the reversed form; if they tie, the natural one is the source. I recommended `for (i = 7; i >= 0; i--)` although my own variants tied. Repeated 2026-11-16 (func_002D26A0): skipped the natural form, matched with an explicit `*p--` walk and argued "the pointer must be in the source" from a countdown compile; plain `for (i = 0; i < 4; i++) a[i] = K;` matched (loop.c: "Reversed loop", stores walk down from a[3]). Run this check BEFORE any explanation, every time.
- **Byte stores force a global-pointer reload** (func_0015F7E0, 2026-11-08, observed in target and ours): through `G->field`, gcc reloads `G` after every `sb` but not after `sw`. Group the target's stores by which reload of `G` they use: each group is one source run, word stores first and its single byte store last. Recovered a 52-store block's order (143 → 44 differing) in one step.
- **Small verified shapes from func_001596A8 / func_0015F7E0 (2026-11-08, all tested in scratch):**
  - **No unrolling at -O2:** a constant 3-pass loop stays a loop (212–218 words vs 204 straight-line); straight-line target code = straight-line source.
  - **Multi-array loop store order:** in `for (i…) { a[i]=0; b[i]=0; c[i]=0; }` gcc's strength-reduced pointer starts at the array stored LAST in the body, others as fixed offsets; the target's offset order (+0x3C, +0x28, +0) gives the source order directly.
  - **`if/else` value pick:** temp + one store after → `movz`; target with a branch per arm and a separate `lui` in each arm = a store in each arm, later tail-merged. Ternary / `x == K` → branchless `xori`+`sltiu`.
  - **Signedness shows only in compares:** byte copies load `lbu` either way; `== 0` tests load `lb` for `char`, `lbu` for `u8`. A `(u8)` cast on the test is folded away (equality with 0), so the pointer's own type must be unsigned.
  - **Address-taken locals** (`&fid` passed to a callee) are reloaded after every call, so a `tmp = fid + 1` reused across calls is a real source temp (inlining it: +169 words).
- **sched1 tie-break order + PRE-inserted loop decrements** (func_0011F2B8, 2026-11-09, `.gcse`/`.sched` dumps + haifa-sched.c `rank_for_schedule`; unresolved, parked). ee-gcc uses the haifa scheduler (configure `-DHAIFA`). Ranking: priority, then `INSN_REG_WEIGHT` (sets minus REG_DEAD/UNUSED notes, lower first, sched1 only), then relation to the last scheduled insn, then dependent count, then luid. gcse PRE turns a loop latch's `i--` into `reg = i - 1` inserted at the END of the header block (`PRE: redundant insn ... reaching reg is N`; latch becomes `i = reg`, the `daddu v1,t0` pattern); placed after the last use of `i`, it kills `i` (weight 0) and jumps ahead of a same-priority `x += c` (weight 1). Statement order inside the header can't change this. When an inserted insn's position is the diff, read the `.gcse` PRE lines and the weights before trying source orders.
- **Return/call slot vs assembler (2026-11-11):** the repo assembles with `991111-01`, which never
  fills a slot gcc left empty. If the target has a volatile access (I/O register store, VBlank
  counter) *in* a `jr ra`/`jal` slot and only the slot differs, it is `dtls13010`-assembled code
  (movie, menu library): no C change fixes it; check with `tcident.py asmsigns -v`. If the target
  has `access; jr ra; nop` and your build fills the slot, the access must be `volatile`.
- **Callee declared `void` vs non-void moves registers far from the call** (func_002D9028, 2026-11-14): last 7 words (register picks in unrelated blocks) fixed only by `vsweep pack`'s `ret:` lever. The callee (libmscd `func_00164A50`) is `void` in its own TU, so the original caller had no prototype (implicit int). In the repo: keep it out of shared headers so the caller sees none, comment at the call. Run `pack` early when a near-match has only scattered register diffs.
- **Declaration order sets spill-slot order** (func_002D9028, 2026-11-14): spilled locals get stack slots in declaration order (table 0x140, hddEnabled 0x144, path 0x148 needed table/hddEnabled/path declared first); register-only locals' order didn't matter. A counter loaded with `lb` and decremented as int must be `s32` (an `s8` local adds sll/sra pairs: 174 -> 98 diff).
- **One address, two symbols: zero-length array before a scalar** (func_002D8D50, 2026-11-15): gcc 2.95 cse folds `(lo_sum (high X) X)` to `X` and reuses any register with `REG_EQUAL X`, so `f(g, &g)` keeps `&g` in a saved register across calls. When the target instead rebuilds the address (`lui`/`addiu`) while reusing the `%hi` for the load, the two are different symbols at one address: `static s32 a[0]; static s32 g;` (zero-size label, same `.bss` offset). Diagnostic that finds it fast: pass any other extern at that address; if that matches, look for the zero-length array. Ruled out on the way: struct/array layouts (base+offset addressing), decl order, flags, block shapes. Needs the TU's `.bss` from C: uninitialized non-static globals are COMMON (placed by the linker elsewhere); statics land in definition order; the TU's `.bss` must fill exactly to the next splat bss subsegment (ld packs the next one right after, no alignment from the `.s`). Also: gcc 2.95 emits an initializer's string literals in reverse order, and an unused `static inline` function still emits its string constants (no code) - used for 2D8658's rodata.
- **Unbounded search loop = explicit pointer walk** (func_002D22D0, 2026-11-16): gcc 2.95 loop.c (`mark_loop_jump`) invalidates a loop that has a jump into it from outside ("Loop at N ignored due to multiple entry points" in the `-dL` dump), and an invalid loop gets no biv/giv strength reduction. So a target that steps a pointer separately from the counter in such a loop needs the pointer in the source (`if (x == *p++) return i + 4;` in `for (i = 0;; i++)`); 180 indexed forms failed. A bounded `for (i = 0; i < 16; i++)` loop is reduced by gcc itself (`table[i]` becomes a pointer). rtl.py's cut dumps drop loop.c's report: compile with `-dL` in scratch and read the raw `.loop` file.
- **variant.py MATCH + full build 8 bytes short** (2D2258, 2026-11-16): a C string literal now precedes an asm-owned jump table; splat emits jtbls `.align 3`, ee-gcc `.align 4`. Fix: `jtbl_X = 0x...; // type:jtbl align:0x10` in symbol_addrs (splat passes it to spimdisasm). Always full-build after a match that changes .rodata ownership.
- **Init position after a call fixes both regalloc priority and prologue order** (func_002D2E10, 2026-11-18, `rtl.py --priority` + variants): `count` (refs 4, live 37, prio 2162) beat `var_22` (refs 12, live 188, prio 2093) for s6 because the `0xFF` init at function entry made `var_22`'s live range long. Moving the inits after the first call (`r = scePadInfoAct(...); a = 0xFF; b = 0xFF; if (r) {...}`, logic unchanged) shortened it and flipped the allocation. In gcc 2.95 a CALL_INSN does not end a basic block, so sched2 hoists those constant loads into the prologue; their source position (after the `jal`) decides where they interleave with the register saves. Signals: a near-match whose only diffs are a register swap between a long-lived local and a count/temp, plus prologue `sd`/`addiu` order. Check `--priority` numbers first; a close priority gap = a lifetime or ref-count lever, not declaration order (21 orders tied). Adding a ref instead (`var_22 <<= 8`) flipped the allocation but left an in-place `sll` and the prologue order wrong. Same function: `for (...; i < count, mask == 0xFF; ...)` (comma operator, user's m2c translation) silently drops `i < count`; the target's mask-test-then-`i++`-then-`i < count` bottom is a plain `for` + `if ((a | b) != 0xFF) break;`, while `&&` in the condition breaks loop rotation (88 vs 77 words).

**Tail calls (2026-11-19, TkGetAsia / TksceGsSyncPath):** ee-gcc 2.9 emits a bare `j callee` only when
the last call is a plain statement; `return f();` keeps a frame (jal + jr). Target `j f` = `void w(void)
{ f(); }` (or non-void without return); framed = `return f();`. Byte-typed params kept across a call
add an extension (TkSetEventNewGame needed s32). m2c also invents args from %hi registers live at a
jal (TkInitScene's func_001BE3B8(&D_..)) and drops pass-through a0 args (TkMsImportantClear).
- **Repeated getter calls = separate variables, not one reassigned local** (TkInit func_00125E40, 2026-11-21): each `jal TkGetFFXCofingAdrs` is real (external calls never merge). One `config` reassigned at each call: 29 words off (one pseudo, one hard reg across all live ranges); 3 locals, block-scoped locals or `static inline` helpers (one call each) all match. User prefers the inline helpers (no point calling a getter twice in one function). Plain `static` isn't inlined at -O2 (would leave a standalone copy); confirm with a full build.
- **Switch/compare on an enum-returning call is unsigned** (same function): a `GameRegion` (no negative values) gave `sltiu`/`beqz` where the target had `slti`/`blez`; `s32` return on the getter (or an `s32` local) fixed it. Also: `x > 0 && x < 7` always folds to `addiu -1`/`sltiu 6`; the target's two branches came from a `switch` on cases 1..6 (Troubleshooting 10, converse).
- **gcc reverses simple array loops: read count-down asm as an ascending loop** (user: "both you and the tools really need to account for gcc's loop reversal", 2026-11-21). Shape: pointer = base + K, counter = n-1, body store then `addiu ptr, ptr, -stride` / `addiu cnt, cnt, -1`, `bgez cnt` (two padding nops). Source is `for (i = 0; i < n; i++) a[i] = ...` with the array STARTING at K - (n-1)*stride (TkInitSaveWork: K=0x1E8, n=56 -> array at 0x10C; TkInit: K=0x18, n=8 -> 0x11). Stored counter value = `a[i] = i`. Writing the loop descending scored worse (25 vs 22). Before proposing struct layouts from a loop, compute the start this way; structfields/addfields now list loop-walked pointers as `[i]` (Tools.md).
- **VU0 inline-asm leaves** (func_001AC068 file, 2026-11-21, 12 full-build matches): one `asm __volatile__` block per function, Sony libvu0.c syntax (`vf4`, `ACC`, `Q`, broadcast in the mnemonic; `vdiv Q, vf0w, vf5x` keeps fields). ee-gcc emits no `.set noreorder`, so gas moves the block's last insn into the `jr ra` (or C `if` branch) delay slot: a cop2 insn there is NOT a hand-written sign. `mfc1 v0,$f12` before the block = float passed as `"r"(t)` (Sony's `"f"` + `mfc1 $8,%n` sits inside). Pointer+offset operands (`"r"(v[5])`) give gcc's `addiu` run. Ref: ref/globe/g272_trunk/ee/src/lib/vu0/libvu0.c.

**Copy-then-mask vs masked initializer (2026-11-22, func_002D2890; another agent's full-build match, mechanism not traced by me):**
`u16 current = curr; current &= mask;` (or a by-value parameter masked in place, `curr &= mask;`) kept a
later `andi 0xffff` on a `u16` accumulator. `u16 current = mask & curr;` let combine drop it (11/37 words).
When a zero-extend disappears because gcc "knows" a value is narrow, try narrowing the variable in place
or moving it into an inline parameter before reaching for barriers. I tested 30+ variants, but every one
narrowed in the initializer.

- **variant.py traps (2026-11-22):** `--proto NAME=...` rewrites the C file's own declaration too, so it
  can hide a per-TU prototype that is the actual match lever (rndt: file-local `s32
  TkGetTotalSyncVCounter()` matched, header s64 adds dsll32/dsra32); compare with and without --proto.
  `-e` on an INCLUDE_ASM line fails ("edit not found"): use `-f` files that keep the func_ name.
- **`lq $8; jr; sq $8` = Sony-style inline asm block assembled by `dtls13010`** (func_001AF188, 2026-11-23). One `asm volatile("lq $8,0(%1)\n sq $8,0(%0)" : : "r"(dst), "r"(src) : "$8")` (like libvu0's sceVu0CopyVector, which uses $6); `-01`'s `as` never moves the `sq` into the `jr` slot, `dtls13010`'s does (full build OK with 1ACEA8.o assembled by dtls13010). Before inventing register-variable/volatile levers for a slot difference, try the other assembler (Toolchain.md "Assembler map"). func_001238B0 (main, `$9` for the dest) still unexplained.
- **Unused initialised local still shifts regalloc** (MsBtlListFieldNum, 2026-11-24, verified by variant): a leftover `BtlListField* field = &list[i];` that nothing read cost 4 words (a0/v0/v1 swapped); deleting it matched. After a refactor, delete the old locals before judging a form ("I tried that, it was worse").
- **Packed variable-length records: read the count before moving the pointer** (MsBtlListGroup, 2026-11-24, verified by full build): `n = p->count; p += 5; p += n*2` (or `entries = p->entries; p = (Group*)&entries[n]`) matched; every one-expression step (`p += 5 + p[0]*2`, `&p->entries[p->count]`) was 2 words off (target adds the header size first). Struct vs byte types gave identical code.
