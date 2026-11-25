# Declarations, constants and structs

## Prototypes and externs

- **Per-TU codegen evidence beats header rules.** When codegen shows how a file declared something
  (call-result register, signedness, a missing prototype), reproduce it in that file even if it
  means a local declaration. read_debug.c called `TkSignalSemaBGThread` undeclared (v1 after the
  call) while libmscd.c saw `void`.
- **A prototype that contradicts the real one stays local.** `func_00117E28` (Switch `Sg_FadeInW`,
  void) matched in 1ACEA8.c only as `s32` (the TU probably called it undeclared). Putting
  `s32 func_00117E28(s32)` in `extern.h` would break the function's own definition once matched and
  change other callers' codegen. Local prototype + lead comment; the header gets the true type later.
  `tidy_externs.py` doesn't know this: check its placement for such cases.
- **Fallback placement map** (no codegen evidence):
  - called only inside its TU → no header entry (a local prototype only if a C use precedes the
    definition);
  - called from other TUs → the owner's area header (`battle.h`, `atel.h`, `include/lib/sasa.h` for
    lib-to-lib);
  - owner unknown or unsplit → an `extern.h` area group (a holding area, never a destination: move
    groups out when the home exists).
- **Check callers before adding a header entry:** `grep -rlE "(jal|j) +NAME\b" asm/jp_intl` plus
  C uses (`symuse.py NAME`).
- **File-local data externs** go in one block at the top of the .c (sorted by section, then
  address), never scattered above functions.
- **Return types from evidence:** callers' use of v0 decides `void` vs a value for wrappers; Switch
  types decide the rest when codegen is neutral (`u8 MsGetBattleEncount` from Switch `byte`, where
  `u8`/`u32`/`s32` all matched). Parameter order follows Switch when codegen can't tell (floats and
  ints use separate registers on the EE: `MsBattleEncountExe(f32 dist, s32 fieldId, s32 groupIndex)`).
- **Header prototypes hold only what is used;** constants are the exception (below).

## Constants

- **Existing enums/defines first:** grep the area headers for every magic number by field before
  inventing one.
- **Value sets → `typedef enum`** in the domain header; derived masks and `X_START`/`X_END`/`NUM_X`
  ranges stay `#define`s.
- **First member of a set needed → add the whole set** (SDK groups copied with real names from
  ref/globe or sce-sdk headers). Prototypes and structs stay used-only.
- **Used in one function only → a `const` local** at the top of it; used across a TU → a file
  define next to the TU's prototypes; across TUs → the area header.
- **Prefix defines with their area** (`ENCOUNT_ROLL_DIST`, `ENCOUNT_CHANCE_SCALE`).
- **Spell derived values from their parts** when it matches: `~(1 << 0x0D | 1 << 0x0E)` over
  `~0x6000`, `count * ENCOUNT_CHANCE_SCALE` over `<< 8`, `& (ENCOUNT_CHANCE_SCALE - 1)` over `& 0xFF`,
  `ALIGN(…, 64)` over a mask. Compile each; offer them routinely.
- **A define keeps the literal's type.** `ENCOUNT_ROLL_DIST 10.0` must stay an unsuffixed double
  (the soft-float calls depend on it, Troubleshooting case 20); an int define in a float context
  (`unk114 *= ENCOUNT_CHANCE_SCALE`) folds to the same constant.
- **Literal case is clang-format's** (`NumericLiteralCase`): don't hand-convert `F`/`f`.

## Structs and fields

- **Offset tag on every member** (`/* 0x0106 */ s8 encountRate;`), gap arrays too
  (`u8 pad106[0x108 - 0x106]`). No `size:` tag on structs ending in a `[0]` array.
- **Splitting a gap:** replace the pad with the members the matched code reads, keeping the
  following offsets; split an array member (`unk1[3]` → `unk1[2]` + `rate`) when one element gets a
  meaning.
- **A `D_` extern that m2c declares inside a declared struct global is that struct's field**
  (func_001AD0E8: `D_00316F96` = `D_00316F70.unk26`). Run `make overlaps` after struct or global
  changes; it lists C globals whose ranges overlap.
- **Switch offsets agree with PS2 only up to the first pointer or 8-byte-aligned member** (64-bit
  build): use Switch `ptr[0xNN]` as a hint for which field, not as an offset.
- **Placeholders say so in chat:** `extern s32 D_00319150[]; … [0x3D14 / 4]++` stands in until the
  save block is sized; a field-based form that also matches is the eventual target.
