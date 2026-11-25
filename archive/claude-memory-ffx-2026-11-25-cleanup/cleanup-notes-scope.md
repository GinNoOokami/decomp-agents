---
name: cleanup-notes-scope
description: "FFX/Cleanup.md holds only project-wide \"closer to the original\" items; per-system cleanup gets its own notes; no small style nits"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 6576646c-edae-40f4-b193-98ecdbb9a21b
  modified: 2026-10-30T01:35:08.779Z
---

`DecompNotes/FFX/Cleanup.md` is for engine-level/general correctness items only (shared macros, build behaviour, common types) that make matching source closer to the original's form. Cleanup for one system belongs in that system's own notes. Small tidy-ups (parameter types, indentation, a repeated expression) are not recorded at all — mention them in chat.

**Why:** The user doesn't want it to become "a dumping ground for every little cleanup note".
**How to apply:** Before adding an item, ask: is it project-wide and about faithfulness to the original? Otherwise route it to the system's notes or leave it in chat. See [[keep-working-log]], [[decomp-notes-docs]].
