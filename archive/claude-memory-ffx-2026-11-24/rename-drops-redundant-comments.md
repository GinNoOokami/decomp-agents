---
name: rename-drops-redundant-comments
description: "When renaming a field/variable, delete its comment if the new name already says it; keep only comments that add information"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 2813f543-3ed4-4dbc-a4c1-c910159285db
  modified: 2026-11-19T05:04:33.520Z
---

When I rename a field (or variable), remove its old comment if the new name now carries the same meaning (e.g. `repeatInterval; // repeat interval?`). Keep the comment only if it holds something the name doesn't (e.g. `repeatDelay; // first-repeat delay?` keeps "first").

**Why:** user, 2026-11-18, after I renamed unk94/unk96 and kept the lead comments verbatim: redundant comments are noise once the name says it.

**How to apply:** part of every rename (struct fields, locals, globals), including comments the user wrote ([[user-comments-are-theirs]] doesn't block this; they asked for it). Mention which comments I dropped or kept. Related: [[lead-comment-shape]], [[cleanup-checklist]].
