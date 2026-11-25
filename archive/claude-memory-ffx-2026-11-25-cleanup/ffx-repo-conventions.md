---
name: ffx-repo-conventions
description: "Conventions agreed for the user's own ~/dev/ffx repo - VERSION var, per-version output, address-named files, toolchain fetch, symbol provenance lines"
metadata:
  node_type: memory
  type: project
  originSessionId: f9744cf8-80ad-4ec4-ae82-43f7bb0923b8
  modified: 2026-11-20T01:48:28.692Z
---

Agreed with the user on 2026-10-28 for `~/dev/ffx` (splat 0.50.1, config `config/jp_intl/SLPS_250.88.yaml`):

- `VERSION ?= jp_intl` in the Makefile; all generated output goes under the version.
- Unconfirmed files are named by start address (`battle/1AC068`) and renamed only with evidence.
- Toolchain: fetch script puts it in ignored `tools/cc/`, from the local cache `~/opt/ee-toolchains/archives` or decomp.me, sha256-checked.
- `symbol_addrs.txt` (hand-authored; the generator `symgen.py` was retired 2026-11-10 at the user's request, never regenerate it; add entries by hand): **Update 2026-11-05 (user):** inline text after the attributes is fine (splat 0.50.1 skips words without `:`; a word with `:` is parsed as an attribute and errors unless it is type/size/rom/segment or a boolean flag). Names recovered from original sources get an inline provenance note, e.g. `task_wait = 0x0017133C; // type:func original name, from the PC port's placeholder string ...`. Library names sit in blocks under one evidence header per kind (libgcc, newlib, newlib placed by object, kernel stubs); game names get NO comment in symbol_addrs (user, 2026-10-31: "that's what headers and code are for" — evidence goes in the header/code where the name is used); odd cases only when needed; the unnamed start list has one header (user, 2026-10-29: wanted the grouped headers kept, not dropped).
- Directories (2026-10-29, user decision): our own system-based directories are fine (flat is unmaintainable, and the binary rarely names directories); keep an original directory when `__FILE__` gives one (`src/main/`, `src/chr/`, `eiProg/`). File names stay evidence-based (original name if known, else start address).
- `make` must still pass (load image sha1 check) after each change.

**Why:** the user built the repo from scratch to learn the setup; these were decided together and shouldn't be re-litigated.

**How to apply:** follow them in every repo proposal; still ask before editing (see [[ffx-start-here]], [[repo-self-contained]], [[readme-upkeep]]).

**SDK headers (user's rule, 2026-10-30):** our own minimal headers in `include/sdk/` (included as `<sdk/libcdvd.h>`) named after Sony's (`sifdev.h`, `libcdvd.h`, …), declaring only functions FFX is shown to call, with the evidence and address in comments; append as more are confirmed. Sony's headers (GirianSeed/globe repo, local copy `~/dev/DecompNotes/ref/globe/{g20,g242,g251,g272}_trunk` = shallow clones, one per release branch, git-ignored; headers are Shift-JIS, so use `grep -a`/`iconv`; "SCEI CONFIDENTIAL") are reference only — check them before answering SDK constant/prototype questions from memory, never copied; prototypes follow 2.4.2 unless FFX's code shows otherwise. **Constants (user, 2026-10-31):** when code uses one value of a Sony `#define`/enum group, add the whole group's values to our header up front (e.g. all of libcdvd's "Interrupts" codes when `SCECdComplete` is used); functions are still added only once FFX is shown to call them. Apply it in cleanup proposals too: naming one flag (e.g. `SCE_TRUNC` for an sceOpen mode) means proposing the whole block; missed once on 2026-10-31 (func_001656E0) and the user had to remind me. Format with `.venv/bin/clang-format` (ColumnLimit 120). In `symbol_addrs.txt`, evidence comments only on names with direct evidence from the original (assert/printf strings, PC placeholders: "original name, from …"); our own names get no comment, even ones styled after originals (user, 2026-11-05, `task_run_asm`).

**Header/source layout (user, 2026-11-03):** `include/sdk/` = SDK, `include/lib/` + `src/lib/` = the original `lib/src` library (libmscd, read_debug, decode, 255928.s, sa_task.h), `include/game/` = game-specific headers (none yet), `include/extern.h` = interim declarations for unsplit owners, in sections by area; move a section out once its owner is split. **Update (user, 2026-11-19):** "A generic battle.h header is still better than extern.h for known mapped regions": a region already mapped to a system (SystemSplit.md, e.g. battle core) gets a generic area header now, not after its split; extern.h only for owners whose system is unknown. Planned extern.h pruning after tklib.c. **Hand-written asm never includes headers (user, 2026-11-05):** its declarations exist only for C callers, so they go in the callers' area header now (task asm → `sa_task.h`), not "once split"; extern.h keeps only asm whose callers have no area yet (`SPAD_CALL`, `sa_printf_set`). Layout options (lib/src+lib/include, sasa/) kept in DecompNotes FFX/ProjectLayout.md.

**Headers per area, not per TU (user, 2026-11-04):** "It's not clear that one TU has one header" — the
three movie files share one `include/lib/movie.h` (sections per file inside). `tklib.c` got
`include/common/tklib.h`. **Upper TK layer (user, 2026-11-10):** files at 0x2D19F8–0x2D9538 go in `src/common/` (address-named, prototypes in `include/common/tklib.h` with per-file sections); subfolders like `common/audio`, `common/io` maybe later, not now. **Folders from PC paths:** keep only what reads naturally in our one tree
(`..\ps2lib\common\tklib.c` → `src/common/`, user's call); record such choices in ProjectLayout.md.

**`game/` = all game-specific code (user, 2026-11-21):** our own classification, not an original
folder: everything that isn't explicitly lib/SDK/kernel code (main, chr, atel, battle, game state…)
goes under `src/game/` / `include/game/`, with original folder names (`chr`, `main`) as subgroups
where known. Don't propose moving game code out of `game/` to mirror the original tree (I suggested
`include/chr.h` beside `mem.h`; wrong). Existing outliers: `src/main/util.c`, `include/mem.h`.

**Types (user, 2026-11-22):** game code uses our `common.h` types (`f32`, `s32`, `u32`, …) wherever possible, never `float`/`int` (applied to 1AC068.c). SDK types (`sceVu0FVECTOR` etc.) stay as Sony names them. Use these in every proposed or matched function.

**Literal suffix case comes from .clang-format** (`NumericLiteralCase: Suffix: Upper` → `3.0F`, 2026-11-25).
User said they prefer lowercase, then remembered clang-format sets it; follow whatever the config
says, don't hand-convert.
