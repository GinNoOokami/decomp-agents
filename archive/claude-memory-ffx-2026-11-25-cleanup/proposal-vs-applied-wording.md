---
name: proposal-vs-applied-wording
description: Result messages must say plainly which changes are proposals for the user vs edits I made; no unrelated side remarks (memory notes) at the end
metadata:
  node_type: memory
  type: feedback
  originSessionId: e47fce8b-d970-47b5-827e-e68f211a06a7
  modified: 2026-11-24T07:50:54.510Z
---

When reporting a match found in scratch, say "for you to apply" explicitly and quote the user's
current code next to the replacement. Never end with a side remark about my own files (e.g. "I also
added the lesson to my notes"): it reads as an edit to their tree.

**Why:** 2026-11-23 (func_001AE3D0) I wrote "your tree is untouched", then a numbered "four changes"
list and "I also added ... to my matching notes"; the user read it as me claiming to have applied
changes without explaining them ("I'm confused"). They had already applied 3 of the 4 themselves.

**How to apply:** re-read their current function before writing the result; list only what is still
missing vs their version, with the reason (what gcc does) in the same message. Memory/notes updates go
unmentioned unless asked. Related: [[ffx-matching-skill]], [[cite-sources-and-runtime-checks]].
