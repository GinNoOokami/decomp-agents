---
name: scope-answer-is-not-go
description: "Answering a scope question (\"do it all in one pass\") about a noted TODO is not a request to start it now; finish the current function first"
metadata:
  node_type: memory
  type: feedback
  originSessionId: bfce77a3-d28f-47f2-bbb6-51d5e4f9d558
  modified: 2026-11-19T21:17:51.486Z
---

When the user asks me to note a TODO and then answers my scope question ("might as well do it all
in one pass"), that decides HOW the later pass will go, not WHEN. Keep going with the function in
progress; the TODO waits until they ask for it.

**Why:** 2026-11-19, the libmscd → Switch rename pass: I started pairing right after their scope
answer, in the middle of TkCdReadWithMalloc. User: "I didn't mean do it right now" / "At least
let me finish the current function first, geeze".

**How to apply:** after a scope answer, confirm it in one line ("noted for later") and return to
the current function. Start big side passes only on an explicit "now"/"let's do it". See
[[usage-budget]], [[ffx-repo-workflow]].
