---
name: logic-first-then-match
description: "While the user is translating m2c into real C, review only structure and logic; register/stack/placement diffs wait until the end"
metadata:
  node_type: memory
  type: feedback
  originSessionId: efe8d60a-fbc1-4f79-84ff-a38bf4cca4fc
  modified: 2026-10-20T22:59:14.888Z
---

When the user asks "how does block X look" while they're still turning m2c into real C, check behaviour and control-flow structure against the asm/m2c only. Leave out register allocation, stack-vs-register locals, block placement and cross-jump differences until they say they're fine-tuning the match.

**Why:** said on 2026-10-20 (BattleActionType0C): "Fine tuning the match can wait until the end, I just want to make sure the structure and logic matches in this phase." My per-block reports had mixed codegen nits (sp10 on stack, error-call placement) into logic reviews.

**How to apply:** in this phase, say whether the block behaves like the original and has the same branch/loop structure. Keep codegen differences in my own notes and bring them up when the user starts matching. Always state which kind of check I ran (see [[scratch-match-workflow]]).
