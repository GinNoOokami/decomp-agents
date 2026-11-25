---
name: tools-lean-evaluate-first
description: Keep FFX tools lean; propose refinements, park bigger ones in ToolingIdeas.md, let the user evaluate before building
metadata:
  type: feedback
---

The user wants tools to stay small and wants to use a tool for a while before it grows. On
2026-11-06 they declined a `--ref` option and a `ret:` tag for nextfunc.py, deferred the m2c-gap
feature ("evaluate what we have before making too many big changes"), and asked for tidy_externs
comment handling to go to ToolingIdeas.md instead of being built. They did accept small,
clearly useful additions (magic-constant count, graded globals, `--by`, unref group, bug fixes).

**Why:** big features add maintenance and token cost before their value is known; real use shows
which refinements matter.

**How to apply:** offer a refinement in one or two lines with its cost; build it only on a yes.
Anything bigger than a small option: write it up in `DecompNotes/FFX/ToolingIdeas.md` (what,
evidence, caveats, when to revisit). Bugs the user reports: fix right away, test on a scratch
copy. Related: [[tools-reference]], [[pilot-before-frameworks]].
