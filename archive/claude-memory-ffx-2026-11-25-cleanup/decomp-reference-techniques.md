---
name: decomp-reference-techniques
description: "Reusable techniques learned from other decomp projects (ff9-decomp, giten-decomp, Rozelette/ffx) and where they're written up"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4709b511-81f1-457a-b4ab-149239db6d3c
  modified: 2026-10-28T23:50:11.045Z
---

Full write-ups: `~/dev/DecompNotes/StartingADecompProject.md` Appendix B (giten, MSVC 5.0 SP3), C (Rozelette/ffx, PS2), E (ovltrace runtime tracing), F (ff9-decomp, PS1). Read those before re-researching.

Techniques worth applying (2026-10-28):
- **Accidental-match checks** (from ff9-decomp): void functions whose callers read `$v0`; functions reading more arg registers than declared/passed. Use in cleanup/reviews: C that matches only via register leftovers misstates signatures.
- **Differential testing of a non-matching draft**: run original + draft in a MIPS interpreter from the same random states, compare calls / memory written outside the frame / `v0`. Catches logic errors before matching work (ff9 `tools/equiv.py`).
- **Overlay base by `jal` voting**: candidate base with the most internal `jal` targets on prologues; static complement to ovltrace.
- **Compiler matrix probe**: one scratch file × every candidate compiler/flags, per-function match with relocations masked (filter only; SHA1 decides). Per-range `-O1` shows as `$ra` saved first + unscheduled prologue. Never test compiler hypotheses on SDK library code (built with a different toolchain).
- **GCC jump tables**: `.align 3` emitted vs original 4-byte packing → rewrite to `.align 2` (ff9).
- **Assembler hazard nops** (FFX, 2026-10-28): when compiled code differs only by `nop`s, count hazard patterns across the whole ELF (e.g. `mtc1`→use vs `c.cond`→`bc1`) to learn the original gas's padding rules, then fix with a `.s` rewrite (`.set noreorder` around the instruction). Check the parser keeps bare `nop` lines (trailing newline bug once hid them).
- **Evidence-tagged names** (`[string]`, `[bios]`, `[runtime]`, `[code]`); matching-time comments are leads. Fits [[naming-high-confidence-only]].
- **MSVC 5 data layout marks objects** (giten): definition-order data, string literals after a TU's data, bss ordered by name hash, contiguous per object in link order. Use with [[pc-port-tu-analysis]].
- **Runtime harness**: keep emulator paused, commands run N frames then pause; per-disc savestate library as the main tool. See [[pcsx-redux-debugging]].
Techniques from FFX (2026-10-29; details in FFX notes):
- **Library fingerprinting by compiling its source** with the original compiler, whole objects compared at one base (relocations masked). Headers the compiler package lacks (`stdarg.h`/`va-mips.h`, `limits.h` = `glimits.h`, `float.h` = `config/float-i64.h`) come from the same gcc release's source; the exact match itself validates them. Small functions that match several places, or that two files compile identically, are placed by a unique neighbour of the same object at the same distance.
- **Probe the compiler instead of assuming**: compile a 3-line test for `sizeof`, alignment (`.p2align`), inline-asm epilogues, char signedness.
- **Hand-written asm signals**: starts not 8-aligned (gcc aligns every function), `$at` as a working register, `addu rd,$0,rs` moves, `ra` kept in a register, no compiler epilogue after the code. spimdisasm's `/* handwritten instruction */` only flags instructions gcc never emits and misses these.
- **SDK library extents**: `PsIIlib<name><ver>` ID strings in `.data` plus `.data`-in-link-order users. Kernel-call stubs are named from the number in the code plus a public table, but check the table's shape (which i-variants exist) against the names; tables differ across kernel versions.
- **`.data`/`.rodata`/`.bss` order inside one file** (declaration order) joins functions across a gap; COMMON globals sit at the end of `.bss` and carry no file signal.
- **Names from assert expressions** (FFX, 2026-10-30): asserts that pass `#e` give original names;
  the value tested by the branch just before the call is the subject (global, constant, field).
- **`__FILE__` spelling = build root**: basename vs `src/x/y.c` shows where each file was compiled
  from; `..\` paths in an MSVC port show the project's folder grouping. See [[ffx-pc-port-workspace]].
Avoid for our goal ([[goal-quality-reconstruction]]): one-C-file-per-binary splits, register pinning to force matches.
Techniques from FFX SDK naming (2026-10-30; details in FFX/SystemSplit.md, Toolchain.md):
- **Decode SIF RPC numbering from the disc's IOP modules.** Split `IOPRP*.IMG` by its ROMDIR (16-byte entries name/ext/size, files 16-aligned); in each IRX find import tables (magic 0x41E00000, stubs `jr ra; addiu zero,zero,N` = export N), then the RPC server's dispatch (jump/function table indexed by fno); each handler's one import names the operation via ps2sdk's `exports.tab`. Done for FILEIO and LOADFILE.
- **Sony library objects** build with `ee-gcc2.9-991111-01-dtls13010` + the globe repo's Makefile flags, not the game's toolchain (its `as` fills inline-asm delay slots). libgraph's `graphdev.c` builds one object per `#ifdef SCE_GS_FUNC_NNN`. Bracket the version with two globe branches (2.0 / 2.4.2 / 2.5.1) and let FFX's code decide where they differ.
- **SDK header declaration order = library link order** (libdma, libgraph): a function missing from the nearest source sits in its header slot.
- **`PsIIlib<name><ver>` records the patch a library last changed in**; the highest ID (plus an `IOPRP<ver>.IMG`) gives the SDK release.
- **Alphabetical archive order + exact-match anchors** narrow unmatched libc members to a window, not a name; behaviour (differential tests) must pick the name.
- **FFX file-edge tests (2026-11-04, details FFX/FileBoundaries.md, AlphaBuild.md):** static `.bss`
  shared by two functions = same file (ee-gcc puts only uninitialized statics in `.bss`; only in the
  low block, globals exist higher); `.data`/`.bss` link order (a later object's item can't precede an
  earlier one's); a switch table (`.align 4`) fixes its file's `.rodata` block start to 0 mod 16, so an
  item at 8 mod 16 can't open that block (assembler padding only, linker-independent); identical
  `__DATE__` strings across builds = file not recompiled = its code identical.
