---
name: keep-working-log
description: FFX/Log.md is retired as a running log; write to it only for notable project architecture decisions
metadata:
  node_type: memory
  type: feedback
  originSessionId: 6576646c-edae-40f4-b193-98ecdbb9a21b
  modified: 2026-10-30T00:24:40.525Z
---

`DecompNotes/FFX/Log.md` was a detailed step-by-step log of the bootstrap phase (disassembly analysis, project setup), which the user asked for so they could review how a decomp gets started. That phase is over (2026-10-29). Don't log investigations, tool setup details or function matches there. Add an entry only for an interesting architectural decision about the project (layout, build, toolchain, tooling approach), and keep it short.

**Why:** The user said the log was "getting a bit crazy" with every detail; it was meant as introspection on getting started, not a permanent diary.
**How to apply:** Findings worth keeping still go in the topic notes (Toolchain.md, SystemSplit.md, …) per [[decomp-notes-docs]]; session status goes in the handoff ([[handoffs-not-in-notes]]). Scripts still go in `~/dev/ffx-analysis/scripts/`. Evidence strength rules unchanged: [[state-evidence-at-real-strength]].
