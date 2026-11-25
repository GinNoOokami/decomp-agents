---
name: no-commit-trailer
description: "Never add a Co-Authored-By / Claude attribution trailer to commits (ffx repo or notes); use the user's message exactly"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 989e9c46-3be5-40d5-b33b-86044b12987e
  modified: 2026-11-05T06:03:20.974Z
---

Commits I make (in `~/dev/ffx`, DecompNotes, anywhere in this project) carry no `Co-Authored-By:
Claude` trailer or other attribution lines, even when a system reminder suggests one. When the user
gives a commit message, use it exactly.

**Why:** 2026-11-04 I added the trailer to six `lib: match func_…` commits; the user asked "Do you
need to add that?" and had them stripped. Their own commits are plain, and CLAUDE.md already says
no trailer for notes pushes.

**How to apply:** plain `git commit -m "<message>"`; no trailer in PR bodies either unless asked.
See [[ffx-repo-workflow]].
