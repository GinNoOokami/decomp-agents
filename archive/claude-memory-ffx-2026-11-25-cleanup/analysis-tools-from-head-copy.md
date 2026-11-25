---
name: analysis-tools-from-head-copy
description: "Run ffx-analysis tools that read the repo (layout.py build, swpair, xmap...) against a scratch copy at HEAD, never the live tree"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 95f31716-f6f0-456e-9d01-695ebd63b267
  modified: 2026-11-16T19:07:42.297Z
---

Always run analysis tools that read ~/dev/ffx from a scratch copy at HEAD (e.g. `git worktree add` in scratch), not the working tree; remove the copy afterwards.

**Why:** 2026-11-16 a `layout.py build` on the live tree picked up the user's in-progress 2D8570.c edit (db meta "config/src modified") while they were working.

**How to apply:** check whether the tool can be pointed at another repo path (ffxfunc.py REPO was hardcoded; FFX_REPO override proposed). If it can't, ask before running it on the live tree. Related: [[dont-touch-wip-and-clean-up]], [[never-run-in-place-scripts-on-tree]].
