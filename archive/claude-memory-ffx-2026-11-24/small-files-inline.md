---
name: small-files-inline
description: "Files under ~50-100 lines go straight into the chat, not the scratch + clip.exe handoff"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 48b2336b-2a8e-40ab-ad53-efe697ab3ca1
  modified: 2026-10-29T04:20:59.067Z
---

For small files (under roughly 50–100 lines), paste the content directly in the chat instead of the
global CLAUDE.md "scratch file + `cat … | clip.exe`" handoff. Keep the clip handoff for larger files.

**Why:** the user said so on 2026-10-28 while setting up the new FFX repo (`~/dev/ffx`); the clip
step is overhead for short files they read in full anyway.

**How to apply:** proposals for their code (gitignore, requirements, Makefile pieces) inline when
short. Still don't write into their tree without permission. Exception: Makefile recipe lines pasted in chat lose their tabs when copied from the terminal (make: `missing separator`); offer to apply the change, or use the clip handoff. Python/venv setup needs no explanation
(they know it); Makefiles get explained (less familiar). See [[ffx-start-here]].
