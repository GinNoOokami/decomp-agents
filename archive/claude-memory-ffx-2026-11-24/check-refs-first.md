---
name: check-refs-first
description: FFX split/matching tasks - look in ref/ first (khasinski tu_map.tsv, Rozelette yaml, ref/globe Sony sources) before running our own tools
metadata:
  type: feedback
---

At the start of any file-split or matching task in a range, check the reference material before our own tool passes:
khasinski's `config/tu_map.tsv` + yaml (PAL; file offsets, vram = offset + 0xFF000; same addresses as ours in main),
Rozelette's yaml and matched src, and `ref/globe` / `ref/sce-sdk-242` Sony sources (e.g. `globe/g272_trunk/ee/src/lib/vu0/libvu0.c`).

**Why:** 2026-11-21 retro, user agreed. libvu0.c settled the inline-asm syntax at once, and the user had to point me at khasinski ("Why not check the llm ref repo? It is a lot more complete than rosa's") after I'd spent passes on combine/xmap.

**How to apply:** their edges/names are leads to test with our evidence ([[community-tools-corroborate-only]]); note their confidence column. Related: [[pilot-before-frameworks]], [[usage-budget]].
