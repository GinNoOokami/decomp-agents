---
name: goal-quality-reconstruction
description: "User's goal for any decomp project is quality source reconstruction, not speed or a port; weigh techniques and advice by that"
metadata:
  node_type: memory
  type: user
  originSessionId: 4709b511-81f1-457a-b4ab-149239db6d3c
  modified: 2026-10-28T07:04:33.410Z
---

For the user's decomp projects (ff7-decomp and future ones, PC or console), the goal is **quality source reconstruction**: original structure (TUs, statics, data ownership), correct types and signatures, evidence-backed names. Speed and ports are not goals (user, 2026-10-28).

**Why:** stated when reviewing the agent-driven ff9-decomp, which trades TU recovery for speed (one C file per binary) and adds port-oriented tooling.

**How to apply:** when recommending techniques from other projects, adopt what improves source fidelity (accidental-match checks, differential testing, evidence-tagged names, TU recovery) and flag shortcuts that hurt it (single-file splits, register pinning, volume over understanding). Frame "port" checks as source-quality checks. See [[decomp-reference-techniques]].
