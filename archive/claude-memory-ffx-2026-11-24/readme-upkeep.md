---
name: readme-upkeep
description: Claude keeps ~/dev/ffx/README.md updated as each setup/build step lands; the user reviews and tweaks
metadata:
  node_type: memory
  type: feedback
  originSessionId: 48b2336b-2a8e-40ab-ad53-efe697ab3ca1
  modified: 2026-10-29T06:53:02.653Z
---

The user dislikes writing READMEs and gave standing permission (2026-10-28) for Claude to update
`~/dev/ffx/README.md` whenever a step changes setup, dependencies or build commands. Keep edits
small (a few lines per step), self-contained (see [[repo-self-contained]]), and show them so the user
can review; expect them to tweak wording afterwards.

Content: only what a reader must do (commands, required deps, a gotcha that breaks the build).
No implementation details (2026-10-29: they removed a paragraph on how the toolchain is fetched:
"the first thing people see … no one wants to read through pages of useless implementation
details"). Those belong in Makefile/script comments.

**Why:** they'd rather review than write it; doc drift was already happening (venv order, missing
deps) when they wrote it alone.

**How to apply:** this permission covers README.md only; other files in the repo still need asking
or an explicit request. See [[small-files-inline]].
