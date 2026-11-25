---
name: ffx-start-here
description: "Where the FFX work starts - current goal (split the main ELF into game systems), what ~/dev/ffx is, work folders and tools, notes to read first"
metadata:
  node_type: memory
  type: project
  originSessionId: 8c5ad48d-8d16-4ca4-846e-224a39661a2e
  modified: 2026-10-29T02:50:59.258Z
---

**Since 2026-10-28 `~/dev/ffx` is the user's own fresh project**, started from the ground up so they learn how each piece works (don't scaffold it for them unasked; ask before editing it, as for any of their code). The Rozelette/ffx clone (early FFX decomp by other authors, read at `e3a4f2b`; JP International `SLPS_250.88`, ee-gcc 2.9-991111 `-O2 -G0`, splat) now lives at `~/dev/DecompNotes/ref/rozelette-ffx` (git-ignored): read-only reference, community evidence only (see [[community-tools-corroborate-only]]). Analysis scripts must not depend on it: the ELF is `~/dev/ffx-analysis/builds/ps2/jp_intl/SLPS_250.88` (same sha256 as the clone's copy); their symbol names are copied to `~/dev/ffx-analysis/ref/symbol_addrs.txt`.

**Goal (since 2026-10-28): split the main ELF's `.text` into major game systems** (battle, field, menu, …). Overlay identification is done: the ELF holds all the main code, and the only loaded EE code is 581 effect modules at 0x01B00000. The user found the overlay split less useful than expected because FFX is monolithic. Results: `~/dev/DecompNotes/FFX/Overlays.md`, `Archive.md`, `Log.md` (all evidence). The old plan is `OverlayIdentification.md` (superseded goal). **Current work (since 2026-10-29): C files in `~/dev/ffx`; `decode.c` matched, `dbg/read_debug.c` next (user writes the C, Claude checks with objdiff-cli); split evidence still in `FFX/SystemSplit.md`**; the latest handoff (status + next steps) is `~/dev/DecompNotes/handoff/FFX.md` (git-ignored), read it first. Toolchain/options findings live in **`FFX/Toolchain.md`** (gcc 2.9-ee `991111-01`-era, `-O2 -G0` where tested; toolchains in `~/opt/ee-toolchains`, run only via `~/dev/ffx-analysis/scripts/eetc.sh`, which is allowed in settings; m2c in `~/dev/ffx-analysis/tools/m2c`; objdiff-cli in `~/opt/objdiff`, one-shot function diffs via `~/dev/ffx-analysis/scripts/odiff.py <unit> <func>`).

**Work folder:** `~/dev/ffx-analysis/`
- `disks/`: Redump-verified ISO.
- `scripts/`: archive, unpack, PCSX2 client, logger, cmdtable.
- `out/`: dumps, logs, PC samples per mode, `text.s`.
- `.venv/`: `splat64`, `spimdisasm`, `rabbitizer`.

Ghidra 12.1.4 is at `~/opt/ghidra`, with ghidra-emotionengine-reloaded v2.1.38 installed as a user extension (language `r5900:LE:32:default`). Headless project: `~/dev/ffx-analysis/ghidra/ffx`; scripts in `ghidra/scripts/` (`ExportGraph.java`). Run with `analyzeHeadless <dir> ffx -process SLPS_250.88 -noanalysis -readOnly -scriptPath … -postScript X.java <args>`.

Runtime: PCSX2-MCP on Windows, reached directly over mirrored WSL networking (see [[emulator-scripting]]). The user plays and narrates; their save is early game (around the boat to Kilika/Luca).

**Other FFX builds as analysis inputs:** US alpha (`ffx-analysis/builds/ps2/us_alpha/`, `FFX/AlphaBuild.md`) and the older US
demo from Jampack Winter 2001 (`ffx-analysis/builds/ps2/us_demo/`, `FFX/DemoBuild.md`): the demo keeps `__FILE__`/debug
strings and `__LINE__` values that International lost.
Original JP `SLPS_250.50` (Jun 2001, oldest, older SDK; `ffx-analysis/builds/ps2/jp_orig/`, `FFX/JpOrigBuild.md`) and
PAL English `SCES_504.90` (2026-11-05, `ffx-analysis/builds/ps2/pal/`, `FFX/PalBuild.md`): built Jan 2002, after
International; same ELF as khasinski-ffx. Build problems and fixes: `FFX/Troubleshooting.md`.
All builds live under `ffx-analysis/builds/{ps2/<build>,pc,vita}` (since 2026-11-15); Ghidra projects under
`ffx-analysis/ghidra/` (PS2 `ffx`, `pc/FFX{,-2}`, `vita/FFX`). Vita JP HD Remaster `PCSG00219`: FAGDec ELF output in
`builds/vita/fagdec/` (ARM, ET_SCE_EXEC, no symbols); loader = VitaLoaderRedux built for 12.1.4 (user extension `vlr`).
Switch HD Remaster: game module `builds/switch/nro/ffx_phyreApp_2015.elf` keeps 44k dynamic symbols
(`builds/switch/symbols.tsv`; `FFX/SwitchBuild.md`); our own PS2 pairing is the next step.

See [[decomp-notes-docs]], [[keep-working-log]], [[decomp-reference-techniques]], [[cross-version-analysis]].

The session-start git snapshot can still show the old Rozelette commits (#7–#9, louist103); ignore it and trust `git log` in ~/dev/ffx.

Another reference (2026-11-04): `~/dev/DecompNotes/ref/khasinski-ffx` (khasinski/final-fantasy-x-decomp,
PAL `SCES_504.90`, heavily LLM-made). The user added it for tool ideas and techniques only; summary and
signal comparison in `FFX/RefProjects.md`. Its TU map and names are community claims
([[community-tools-corroborate-only]]).

**Current work (since 2026-11-04 evening): splitting files in the repo toward battle's data
loading** (`FFX/BattleData.md`); splits so far hddcopy, sa_task, tklib (`src/common/`), movie files;
latest state in the handoff. Before any region: `notescheck.py` it.

**Boundary toolsuite (since 2026-11-04):** facts per build in SQLite (`ffx-analysis/scripts/layout.py`,
dbs for jp_intl, us_demo, us_alpha), signal tools with calibration, `combine.py`; refresh all with
`evidence.py`. Current results: `FFX/FileBoundaries.md`; tools: Tools.md "Layout facts". Next planned
work: splitting TU/file boundaries in the repo from those results.
Before proposing any TU edge, run the checklist in FileBoundaries.md "Checks for one TU edge"
(includes `calldir.py` call-direction leads, which never decide alone).

**Work order (user, 2026-11-07):** methodical, outward from the foundation: the shared game lib
(`lib/`, just above the SDK) first, then expand to the code that uses it. Game code (script VM,
battle, …) hasn't been worked on yet. Other projects' game-code names (e.g. khasinski's
`AtelPopStackInteger` for func_0013CEB8) stay leads until we reach that code; they did a broad
low-hanging-fruit pass, we don't.
