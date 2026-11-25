---
name: ffx-matching-skill
description: FFX matching knowledge (working agreement, procedure, verified ee-gcc lessons by symptom) lives in the ffx-matching skill in ~/dev/decomp-agents; load it for matching, hints, reviews, post-match cleanup and type/struct decisions; new lessons go there
metadata:
  type: reference
---

All FFX matching knowledge moved on 2026-11-24 from ~20 memories into the `ffx-matching` skill:
`~/dev/decomp-agents/skills/ffx-matching/` (git, private, remote GinNoOokami/decomp-agents, pushed with
its own deploy key), symlinked into `~/.claude/skills/` and `~/.codex/skills/` so Claude and Codex share it.

**How to apply:**
- Load the skill for matching, matching hints, reviews of translated functions, post-match cleanup,
  and decisions about types, struct layouts or globals' declarations.
- New verified matching lessons go into the skill's symptom file (agent-neutral wording, evidence
  function named), then commit + push decomp-agents (plain message, no trailer). Not into memory.
- Originals before the move: `~/dev/decomp-agents/archive/claude-memory-ffx-2026-11-24/`.
