---
name: start-from-users-current-version
description: "before any matching attempt, snapshot the user's current C for that function and build on it, not on my earlier drafts"
metadata:
  node_type: memory
  type: feedback
  originSessionId: d8b8ba71-d5b2-44ee-a8a4-83ea356c0bf1
  modified: 2026-10-30T21:41:37.097Z
---

When asked to match (or improve) a function, first copy the user's current file to scratch and start from their version of the function, not from my own earlier draft.

**Why:** 2026-10-30, func_00254AD0: I brute-forced from my v2 draft, rediscovering the `offset` local the user had already added; they asked "did you not take a snapshot of my current copy before beginning?"

**How to apply:** `cp src/<file>.c $S/user_snapshot.c` at the start, diff it against my drafts, carry over their changes, and report results as changes against their version. Re-check `cmp` against the tree before saying it's untouched. **When applying a scratch result:** `cmp` the tree file against the copy's base first; if the user edited it meanwhile (2026-11-06: they added comments to movie_init while I tested), re-apply my edits to their current file instead of copying mine over it. Related: [[copy-target-asm-at-start]], [[check-diff-for-current-function]], [[dont-touch-wip-and-clean-up]].
