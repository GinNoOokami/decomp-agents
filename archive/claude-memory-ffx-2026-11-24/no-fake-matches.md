---
name: no-fake-matches
description: "goto/empty do-while(0)/dummy locals that only steer gcc are diagnostics, never a match; look for the real structure (sibling functions) instead"
metadata:
  node_type: memory
  type: feedback
  originSessionId: d8b8ba71-d5b2-44ee-a8a4-83ea356c0bf1
  modified: 2026-10-31T01:14:25.886Z
---

Don't present compiler levers as a match: a `goto` added to force a layout, an empty `do {} while (0)`, a local that only holds a constant, permuter output with unnatural code. Treat each as a diagnostic ("the original's structure differs here") and look for the real shape.

**Why:** 2026-10-30, func_00255158: I reported a "1 instruction off" version built on a goto, an empty do-while(0) explained as a stripped debug macro, and `one = 1`. The user: "These seem like fake matches … they almost certainly didn't have a random goto … if they were concerned with stripping debug code, wouldn't this whole file either not exist or be stubbed out?"

**How to apply:** when a lever closes a gap, say what it tells us about the structure, then find evidence for the real form: sibling or release functions with the same logic (e.g. read_debug.c ↔ libmscd release counterparts), gcc pass behaviour, other matched code. Report scores of natural-C versions as the real progress. Related: [[distrust-unnatural-code]], [[finish-matches-dont-suggest-stopping]].
