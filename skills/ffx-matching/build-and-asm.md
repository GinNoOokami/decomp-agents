# Build, data ownership and assembler issues

## Link and size errors

- **`relocation truncated to fit: R_MIPS_GPREL16`** = a C function's size is wrong, shifting `.data`
  past `$gp` reach (D_0057C200 sits 0x7FF0 below gp 0x5841F0, so ~0x10 bytes of shrink breaks it). See T2 for
  the small-data limit.
- **variant.py MATCH but the full build is 8 bytes short** (2D2258): a C string literal now precedes an
  asm-owned jump table; splat aligns jtbls to 8, ee-gcc to 16. Fix: `jtbl_X = 0x...; // type:jtbl
  align:0x10` in symbol_addrs. Full-build after any match that changes .rodata ownership.
- **Shared rodata literal:** when an asm sibling still uses a string symbol kept by INCLUDE_RODATA, the
  C must pass that extern, not a literal (+16 bytes otherwise).

## Building .bss and .rodata from C

- Uninitialised non-static globals are COMMON (placed by the linker elsewhere); statics land in
  definition order; the TU's `.bss` must fill exactly to the next splat bss subsegment.
- gcc 2.95 emits an initializer's string literals in reverse order; an unused `static inline` still
  emits its string constants.

## Delay slots and the assembler

- **The assembler is chosen per source folder** (Makefile `DTLS_DIRS`, currently `src/lib` and
  `src/game/battle` use dtls13010; the rest 991111-01, which never fills a slot gcc left empty). Only a
  `jr ra`/`jal` slot differs around a volatile access = the code needs the other assembler than its
  folder has: no C change fixes it; check with `tcident.py asmsigns -v` and Toolchain.md's assembler
  map (T16), and raise it with the user as a `DTLS_DIRS`/file-placement question. Target
  `access; jr ra; nop` with your build filling the slot = the access must be `volatile`.
- **Watch item: battle's assembler rests on thin evidence.** `src/game/battle` joined `DTLS_DIRS` on
  one sign (func_001AF188) and none against (Toolchain.md, "Which code used which assembler"); the
  user accepted that to move forward and wants contradictions raised. While matching or scanning
  battle code (including the `Ms*` camera code once split), flag at once: a function that matches only
  with -01 (`variant.py --asm dtls` vs the default), or a target with a fillable `P; jr ra/jal/b; nop`
  slot in a plain function. After a folder is added to `DTLS_DIRS`, recheck with a clean rebuild of
  its objects (they don't depend on the Makefile).
- **`lq $8; jr; sq $8`** = a Sony-style inline asm copy assembled by dtls13010 (func_001AF188), like
  libvu0's sceVu0CopyVector (which uses $6): `asm volatile("lq $8,0(%1)\n sq $8,0(%0)" : : "r"(dst),
  "r"(src) : "$8")`; full build OK with that object assembled by dtls13010. Try the other assembler
  before inventing register or volatile levers. Open: func_001238B0 (main, `$9` for the dest).
- **VU0 inline asm** (func_001AC068 file, 12 full-build matches): one `asm __volatile__` block per function in libvu0.c syntax
  (`ref/globe/g272_trunk/ee/src/lib/vu0/libvu0.c`). ee-gcc emits no `.set noreorder`, so gas moves the
  block's last insn into the `jr ra` slot: a cop2 insn there is not a hand-written sign. `mfc1 v0,$f12`
  before the block = a float passed as `"r"(t)` (Sony's `"f"` + `mfc1 $8,%n` sits inside the block);
  pointer+offset operands (`"r"(v[5])`) give gcc's `addiu` run.
- **gcc's own padding:** `nop; nop` before `sqrt.s`, five `nop`s + `b` for `for(;;);`.

## Compiler quirks

- ee-gcc 2.9 has no `-iquote` (the dir becomes a second input file); its cpp prints a bare `#define`
  for every macro with `-dD`.
- **After a rename, also compile with `-DNON_MATCHING`:** `#ifndef NON_MATCHING` blocks hold C the
  normal build never sees (a compiler-guided rename missed 103 field accesses in func_00163950 /
  func_00255158).
