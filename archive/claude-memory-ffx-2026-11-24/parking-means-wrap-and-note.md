---
name: parking-means-wrap-and-note
description: FFX - "park it" = apply the NON_MATCHING wrap in the tree myself plus the nonmatchings note, using the README's existing status words
metadata:
  type: feedback
---

When the user says to park a function: (1) edit their file myself — restore `INCLUDE_ASM` under
`#ifndef NON_MATCHING` with their current C under `#else` and a comment that starts with the
substance (words off + cause), never "Not matching yet: with NON_MATCHING" (the `#ifndef` says it);
(2) write `FFX/nonmatchings/<ADDR>.md` + README row with status `open` (the only status in use;
don't invent "parked"); commit+push the notes; full build after the wrap.

**Why:** 2026-11-18 (func_002D2890): I only gave a clip file, invented a "parked" status and wrote
a redundant comment opener; the user corrected all three.

**How to apply:** wrap the user's *current* version (re-read first), not my best variant.
Related: [[parked-items-stay-parked]], [[apply-means-edit]], [[finish-matches-dont-suggest-stopping]].
