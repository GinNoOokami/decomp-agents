---
name: splat-c-file-gotchas
description: "splat + old EE gcc gotchas hit when a segment becomes a C file - data/rodata ownership, shared literals, INCLUDE_RODATA, old-as objects, stale asm"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 097ed56a-8827-4283-b2d6-f2206c8d5f47
  modified: 2026-11-10T08:46:06.318Z
---

Learned making the first C files in `~/dev/ffx` (2026-10-29, splat 0.50.1, ee-gcc 2.9-991111-01):

- A `.data` dot-subsegment hands that range to the C object: splat stops emitting it, so the C file must define the data (undefined `D_…` at link otherwise). `.rodata` used by one function is migrated into its `.s` automatically.
- String literals reach m2c only if the C file owns its `.rodata` range. Literals shared by several functions need `rodata_string_guesser_level: 2` (level 1 requires a single reference) and live in `D_*.s` with `INCLUDE_RODATA`, which splat writes only when it creates the C file; add them by hand for existing files.
- Objects from the old EE `as` have a `.symtab` `sh_info` modern `ld` rejects; a plain `objcopy` in place fixes it.
- `.bss` works the same way as `.data`: a `.bss` dot-subsegment needs the C file to define the variables (and `static`, or gcc 2.9 puts them in COMMON, placed elsewhere by the linker). Until then keep data/bss as plain asm segments with cuts at the file's boundaries.
- Every split rewrites `include/include_asm.h`, `macro.inc`, `labels.inc` (`generate_asm_macros_files`, default on) when their contents differ: never hand-format them (`.clang-format-ignore` lists `include_asm.h`).
- objdiff hides a base symbol that has splat's `.NON_MATCHING` alias, so `INCLUDE_ASM` functions show 0% / no base symbol until written in C. Not a config error.
- Exclude `nonmatchings/` from the asm glob; clear `asm/<version>/` before each split (stale files get linked twice); every segment needs a distinct name (same name = files overwrite each other).

- splat writes `nonmatchings/*.s` only for functions still `INCLUDE_ASM` at split time, and our Makefile clears `asm/` before splitting: after a stash/revert to `INCLUDE_ASM`, `make distclean build` (fix idea in FFX/ToolingIdeas.md).
- `size:` on a `symbol_addrs` data symbol makes inner references `sym + off`, so m2c emits `.field`/`[i]` from the context type; C referencing the old inner labels then fails to link. `ignore:true` drops a fake label (e.g. halfword lists read as pointers). `matchings_path` is relative to `asm_path`.

**Why:** each cost a debug round with the user.

**How to apply:** check these before proposing a new C file's YAML/C changes; test in a scratch copy (see [[ffx-repo-workflow]], [[ffx-repo-conventions]]).

**Shared literals while some users are still asm (2026-10-30, read_debug.c):** a C string literal identical to an `INCLUDE_RODATA` symbol (e.g. `"read_debug.c"` = `D_00573400`) is emitted a second time and shifts `.rodata`. Until the last asm user is matched, declare `extern char D_00573400[];` and pass that; afterwards drop the `INCLUDE_RODATA` and use literals (gcc merges identical literals per TU). `.rodata` is emitted in file order, so the `INCLUDE_RODATA` line must sit between the functions whose literals come before and after it.

**Jump tables when asm functions move into a C file (2026-11-01, libmscd):** undefined `.L<addr>` refs from an asm `.rodata` file = jump tables left in plain asm `.rodata` while their functions are now `INCLUDE_ASM` (old EE `as` doesn't export the `jlabel`s). Fix: a `.rodata` dot-subsegment for the C file over just the tables. Owning the file's whole `.rodata` also needs literals for strings used by a single C function. Case write-up: `FFX/Troubleshooting.md` (the user's case-study log; add future build problems there).

**`.data` owned by a file whose users are still INCLUDE_ASM (2026-11-10, common/2D8570, user hit the link error from my untested proposal):** the C definition must be a global with the exact `D_…` name the asm uses (a `static` with another name links nothing; gcc drops unused statics), and every inner label splat made inside the object (e.g. `D_00565B5C` = +0xC) needs a `size:` entry on the base symbol so the asm says `base + 0xC`. Always build a proposed split in a scratch copy (`rsync --exclude build --exclude .git`, full `make`) before giving it.

**Code with no symbol, moving into a C segment (2026-11-10, common/2D19F8, caught in scratch):** (1) a function whose address is only taken (callback, no `jal`) gets a `D_` label, and splat emits it as `alabel` → `.aent` inside the previous function; the old EE `as` rejects `.aent`. (2) Functions nothing references at all get no `.s` in a C segment, so their bytes silently vanish and everything after shifts (showed as gp-relative "relocation truncated" in text.s, not as a missing function). Fix for both: `func_X = 0x…; // type:func` entries plus their INCLUDE_ASM. Check: each new `.c.o` `.text` size in the map = its yaml range.

**symbol_addrs.txt (2026-11-05):** splat parses `key:value` tokens anywhere after `//`, so a colon in a comment ("originals: the PC port…") fails the split with "error reading symbol_addrs.txt, line N". Keep colons out of comments. Renaming an auto `D_…` data symbol in C (e.g. `disc_check`) needs its own `name = 0x…; // size:0x1` entry, or the link fails on the undefined name.

**Symbols outside every segment (2026-11-09, devkit 0x02004A1C…):** a plain symbol_addrs entry links (splat writes it to `undefined_syms_auto.txt`) but prints "Unable to determine a segment", which splat's source marks to become fatal in a future version. With `absolute:true` the warning goes but splat no longer defines the symbol, so the link fails. The fix the user adopted: `absolute:true` plus a hand-written `config/<ver>/undefined_syms.txt` first in the Makefile's `UNDEF_SYMS` (and a prerequisite of the ELF). Constant-address C (`*(int *)0x…`) doesn't match: gas expands it through `$at`.
- **Alignment gaps between objects** (2026-11-14, 2D8E28 .data): yaml `subalign: 8` can't reproduce a following library object that starts 16-aligned; a splat `pad` subsegment is the lever (needs a name: the repo's tools/objdiff_config.py reads sub[2]); the pad is emitted after the linker already aligned, so size it from the aligned address (unfinished, see handoff).

**Empty string literals stay externs (2026-11-20):** splat keeps a function's `""` as a data symbol
(8 zero bytes, e.g. D_00569F20), so writing `""` in C matches the function but the full build is 8
bytes longer (duplicate). Declare `extern char D_xxx[]; // ""` like util.c's D_005699E8.


**Moving a TU's .data/.bss into C (2026-11-21, tklib):** (1) splat `pad` sizes from ROM offsets, so
it can't be used inside the zerofill/.bss group; for a gap before a more-aligned next object, start
the next asm bss segment at the 8-aligned end address so its `.space` holds the gap. (2) Every
false pointer into the new C range breaks the link: `ignore:true` each one. Scan data asm for
`D_` labels in the range first, but words inside text-segment tables only show at link time
(`TkBattleMusicBuffer + 0x625`). (3) Linker symbols that fall in the range (`_gp`) become
`absolute:true` + undefined_syms.txt. ee-gcc: statics in declaration order, arrays/structs
8-aligned, explicit `= 0` stays in .data, unused statics are still emitted.
