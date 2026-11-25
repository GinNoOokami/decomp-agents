---
name: check-diff-for-current-function
description: "when the user asks about \"this\" function without naming it, read their working-tree diff first to find which one"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

When the user asks about a function without naming it ("this one", "what is sp18?"), run `git diff` on their tree first. The function they're working on is the one with m2c output in the diff. Don't guess from asm greps or candidate lists.

**Why:** I once answered about func_801B0000 (picked by grepping asm for 0x18($sp)) while the user was working on BattleActionType16, which was sitting in their diff. They expected me to pick it up from the diff.

**How to apply:** check `git status`/`git diff` before answering any unnamed-function question. m2c names like `sp18` are stack offsets in *their* C, so search the diff, not the target asm.
