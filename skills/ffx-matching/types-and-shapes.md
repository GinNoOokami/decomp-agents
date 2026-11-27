# Types, expressions and data shapes

## Unnatural code means a wrong type

- **C nobody would write = wrong types or layout.** `(&atk->cameraSingleID)[(flags >> 3) & 1]` was a
  `u16 cameraIDs[2]` (FF7, identical code). Test the natural layout (array, union, merged struct),
  present that, and list the other users of the fields that would change.
- **Magic numbers are derived values** (func_00171A88): `n * 0x1310`, `+ 0x607`, `& 0xFFFFFFC0` were
  `ALIGN(0x5C8 + n * sizeof(A) + n * sizeof(B), 64)`. `sizeof` makes the sum unsigned, which gave the
  mask and kept the terms apart. When a cleanup "needs" odd locals, a split shift or literal masks,
  suspect a missing type property (signedness, pointer vs int) first; cleanup isn't done when every
  removal fails, but when it reads like what the programmer typed.
- **Layouts may be padded to the hardware on purpose** (user's working hypothesis, 2026-11-26). The
  original authors align to the EE's 64-byte cache line everywhere (`Mem_SetAlignment(0x40)`,
  `ALIGN_64`, `ROUND_UP`/`ROUND_DOWN(x, 64)` in battle/tklib/movie code; DMA needs 16). An unread gap
  ending on a 0x10/0x40 offset, or a struct size rounded to 0x40, may be deliberate padding or an
  aligned sub-buffer, not unknown fields: check the allocation's alignment and the gap's end offset
  before inventing fields.
- **Fields of B addressed off A's base register = one object** (A extended or a parent struct); gcc
  can't produce cross-symbol offsets from separate externs (FF7 func_800A38FC: `lhu 0x220C($s4)` off
  `&g_BattleWork` landed in `g_BattleSceneContext`; the merged struct matched). Check adjacency
  (A + sizeof(A) == B), test the merged struct; a cast of the base is a labelled stopgap, never
  "needed". Fields of one global through `lw -0x80(a1)` off another field's address = one symbol too.
- **Separate `lui`s = separate globals**; `%lo(sym + 4)` stores with no pointer load = a separate
  global splat has no symbol for (`D_0057C700 + 0x4` was not `D_0057C700->field`; add it to
  symbol_addrs).
- **Every `extern` type of an unnamed global is a guess.** Derive shape from strides, offsets and
  access widths (func_002D1EA0 needed `u8 [1][0x1000]`, a stack pointer stepped by 0x1000; holding
  the old declaration fixed had blocked earlier attempts); include declarations among the levers and
  cite the asm evidence, labelled inferred, when proposing one.
- **`lw` from a global before indexing** = a pointer (`s16* p; p[i]`), not an array.
- **One address, two symbols** (T18, func_002D8D50): cse reuses any register holding `&g`, so
  `f(g, &g)` keeps it across calls; a target that rebuilds the address while reusing the `%hi` = a
  zero-length array before a scalar (`static s32 a[0]; static s32 g;`). Fast diagnostic: pass any
  other extern at that address; if that matches, look for the zero-length array. Struct/array
  layouts, declaration order, flags and block shapes were ruled out on the way.

## Masks, signedness and widths

- **`lui 0xffff; ori 0xffc0` / `ori 0xf800` masks are unsigned operands** (`ALIGN_64((u32)p)`,
  `0xFFFFF800`, `~0x7FFU`, or `~0x7FF` on
  a `u32`; T13); a signed value gives `addiu -64` / `li -2048` (func_00159EA8: 77 words off).
- **Signed power-of-two division** shows as shifts/adds/`movn` (T8): `(x > -1 ? x : x + (D-1)) >> k`
  then `<< k` = `(v + D-1) / D * D`; `& ~(D-1)` gives an `and` instead.
- **Signedness shows in compares:** `lb` for `s8`, `lbu` for `u8` on `== 0` tests (fix the header
  type, e.g. ReadQEntry +0x30); plain byte copies
  load `lbu` either way, and `volatile s8` still compiled to `lbu`, so `lbu` alone is not evidence.
  A `(u8)` cast on a test against 0 is folded away; the pointer's own type must be unsigned.
- **Enum compares are unsigned**; `s32` gives the signed `slti` form.
- **A `u16`/`u8` parameter that is only stored to a narrow field adds an `andi`** before the store;
  declare it `s32` even when Switch says 2 bytes (AtelSetEventElemCharIDFromID: `u16` cost one word).
- **`lbu` + `dsrl` of a flag byte = a 1-bit `u32` bitfield** (func_00158FC8); u8/u64 bitfields don't
  give it.
- **A float constant stored via an integer register** (`lui 0x3F80; sw`) = an integer field holding the
  bits; a `float` field goes through `mtc1/swc1` (func_001581E0).

## Expression forms

- **Cast or conversion around a call result moves the store address before the call** (T11,
  expr.c `expand_assignment`: a bare call RHS is evaluated first, a cast is not a bare call): the
  global's `%hi` sits in a saved register after the `jal`; `rtl.py --diff` shows it before the call in
  the cse pass. func_00162CC8: `D_00595AEC = (Unk00595AEC*)func_0011D370(...)`. Implicit conversions
  count: func_00159668 matched as plain `D_0030D07C = func_00173A98(); if (D_0030D07C & 0x01000000)
  ...` once the getter returned
  `u32` into the `s32` global (a one-use inline helper had matched by accident). A chained
  `G = tmp = call()` that "needs" the temp is the same cause: func_00159EA8 matched plain once
  `D_0058F9DC` was `u8*` instead of `void*`. Read the types, don't keep the temp.
- **Reassociation:** gcc reassociates `x + C - w / 2`; separate `x += C; x -= w / 2;` keeps the order.
  `x -= t - C` becomes `(x + C') - t` and cse folds `x + C'` into a known base (func_00171AB0,
  fold-const.c `split_tree`); target `addiu t,-C; subu x,t` needs `t -= C; x -= t;` or a separate
  variable.
- **`addu` operand order follows the source** (func_00254E08: `index = state->unk20 + id` matched,
  `id += state->unk20` and `id + state->unk20` didn't).
- **Copy-then-mask vs masked initializer** (func_002D2890): `u16 cur = curr; cur &= mask;` (or a
  by-value parameter masked in place) kept a later `andi 0xffff`; `u16 cur = mask & curr;` let combine
  drop it (11/37 words; another agent's full-build match, mechanism not traced). When a zero-extend
  disappears because gcc "knows" a value is narrow, narrow in place or move it into an inline's
  parameter before reaching for barriers; 30+ earlier variants all narrowed in the initializer.
- **Packed variable-length records:** read the count before moving the pointer (`n = p->count;
  p += 5; p += n*2` or `entries = p->entries; p = (Group*)&entries[n]`); one-expression steps
  (`p += 5 + p[0]*2`, `&p->entries[p->count]`) were 2 words off (MsBtlListGroup). Struct vs byte
  types gave identical code.
- **Constant-size `memcpy` is inlined** (T4): a `(src|dst) & 7` test, then `ld/sd` and `ldl/ldr` loops;
  m2c shows unknown `ldl` and invents s64 arguments from a5-a7. Write `memcpy(dst, src, CONST)`.
- **Assert strings name locals** (`"q_cnt != READ_Q_CNT"` gave the loop index `q_cnt`).
- **Direct `arr[i].field` vs a pointer local** (T14): indexing a global folds a word field's offset
  into the base, giving each field its own address register; several copies of one address = direct
  indexing in the source.

## Floating point

- **`fptodp`/`dpcmp`/`dpdiv`/`dpsub`/`dptoli`/`dptofp` calls = `double` literals** (func_001AE4F8): the
  EE has no double FPU, so `x > 10.0`, `x / 10.0`, `x != 0.0` on an `f32` compile to soft-float
  calls; write the literal without `f`. m2c shows them as raw calls with `0x8048 << 0x2F` (10.0's
  upper bits). Only literals that stay in FPU registers (`lui 0x4380` + `mtc1` = 256.0f) are `f32`.
