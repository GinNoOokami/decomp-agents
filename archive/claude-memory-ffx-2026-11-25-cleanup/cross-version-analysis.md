---
name: cross-version-analysis
description: "How to compare builds of the same game (regions, revisions, demos) for TU boundaries and moved code, and how to verify disc dumps against Redump"
metadata:
  node_type: memory
  type: reference
  originSessionId: b49b0d5a-6356-43e1-8484-0eeec9047346
  modified: 2026-11-02T03:03:20.147Z
---

Method (proven on FF7 PS1 battle, 2026-10-15; the FF7 script is `scripts/xver.py` in the ff7-decomp memory dir, PSX-specific):
- Collect function starts from a matching build, then align the other build with relocation-masked exact matches; report where the address delta changes. For distant builds, use opcode-only k-mer voting.
- Also do a **global** exact match + order check: a windowed alignment hides swapped blocks (FF7 JP had two adjacent code+data blocks swapped).
- TU boundaries only show where builds differ (padding shifts, reordering, assembler-style changes); identical regions say nothing. Words that look like padding may be data in a `.s` file.
- Normalise known assembler-version expansions and mask branch offsets before calling two functions different.
- Filter "unique" exact-match anchors to the longest increasing run before aligning the gaps between them: short functions match by chance far away and silently corrupt placements (FFX: a 6-word function moved `meswin.c` 0x2D000 off). Check placements against names the target build still has.
- Learned on the FFX US demo (2026-11-01):
  - **`__LINE__` arguments** at the same call in two builds show where source lines were inserted (+372 at the top of a file = new functions there).
  - **A blanked `__FILE__`** (release `""`) still gives one literal per file. Pair the calls by order and per-file count with a build that keeps the names.
  - **Per-file `.rodata` blocks** work better in the debug-rich build: place its literal refs in the target build, then grow each block from the name string.
  - Don't assume a header's `__FILE__` copy appears once per file; it didn't.

**Redump check:** `curl http://redump.org/disc/<id>/` (search `/discs/quicksearch/<serial>/`); the site refuses HTTPS, so WebFetch fails. ECM images need a decoder: ask the user to install one rather than writing my own.
- Learned on FFX International vs the US alpha/demo (2026-11-05):
  - **A constant address delta across a run** means only that run's own code moved; the step where the delta changes is the size of the insertion (0x1F8 = 63 eight-byte stubs, exactly).
  - **Initial data values per build:** the layout dbs' objcopy images start at 0x100000, so `xxd -s $((addr-0x100000))` on `meta.image` reads a `.data` byte in each build (region byte: International 0, US 1).
  - **Older debug builds keep enum meanings** that release builds dropped: the alpha's host-path switch (`menu/<code>/tuto_savedata`) gave the region codes. Search the older builds' `cstrings` for `%s`/per-variant path families when a selector's meaning is unclear.
- **Which ELF is which:** `ffx-analysis/builds/ps2/debug_disc/elfs/GAMEZ.ELF` is the FIFA demo (symbols, SDK/library boundaries; LibSplit.md), not an FFX debug build. FFX builds (under `builds/ps2/`): `jp_intl/SLPS_250.88` (International), `pal/fs/SCES_504.90`, `us_alpha/SLUS_203.12`, `jp_orig/fs/SLPS_250.50`, US demo via layout db. Check LibSplit.md/DemoBuild.md before calling a file a build (2026-11-14 slip: called GAMEZ a debug FFX build in notes).
