---
name: no-unrequested-decomp-answers
description: "The user does the matching; give hints only when asked, never full or matching C unless explicitly requested (FF7 agreement, applies to FFX)"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 94288afc-c85f-40fe-98d8-d03e9312b519
  modified: 2026-10-30T07:19:05.640Z
---

The user decompiles and matches functions themselves as a learning exercise (agreed in ff7-decomp
2026-10-09; carried over to FFX). Unless they explicitly ask for it, don't hand them decompiled or
matching C, and don't describe what makes a function match. "Matching tips" or "suggestions" for a
function means hints: pointers to what to look at (a type, a branch shape, an asm detail), not a
finished version.

A private scratch reference match is fine for checking that a hint is right; never quote from it
unprompted.

**Why:** 2026-10-30 in FFX I answered "matching tips for func_00254DD8" and "func_00254E08 looks
harder" with complete matching functions. The user: "I didn't ask for a matching version. Don't you
have a record of our working agreement pulled over from the FF7 project?" The FF7 version of this
memory had not been copied to the FFX memory.

**How to apply:** give the smallest hint that unblocks them, verified in scratch
([[state-evidence-at-real-strength]]); offer the next hint rather than the answer. Handoff items like
"next: read_debug.c" are the user's task. Related: [[logic-first-then-match]], [[ffx-repo-workflow]].
