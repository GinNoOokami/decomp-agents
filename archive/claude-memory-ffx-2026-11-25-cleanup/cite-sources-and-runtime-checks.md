---
name: cite-sources-and-runtime-checks
description: Never cite docs/wikis not opened this session; don't call emulator checks "quick" when they need rare game situations (static checks first). Matching-hint rules are in the ffx-matching skill
metadata:
  type: feedback
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX. The
matching-hint parts of this memory moved to the ffx-matching skill ([[ffx-matching-skill]])._

**Citing sources:** never suggest citing "community docs" or a wiki in code comments or notes unless I've actually opened the page in this session. On 2026-10-06 I suggested "script 3 (death counter, per community docs)" from memory, and the user asked "which docs?". Say plainly when something comes from my recall, and offer to look up the real page.

**Don't present emulator checks as "quick" (2026-10-09):** I suggested a "quick emulator check" that needed particular battle situations (cover changes, a hiding enemy). The user pointed out that means finding those situations in a 60+ hour game. Exhaust static checks first (asm scans for direct stores, pointer stores at the field offset, base-address loads, struct copies or clears), and say honestly what a runtime check would need to set up.
