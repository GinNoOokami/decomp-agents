---
name: verify-match-status-from-source
description: "Before calling a function unmatched/asm (or a finding \"open\"), check INCLUDE_ASM vs C in the tree; code comments and older notes go stale"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

Check a function's match status in the source (`grep INCLUDE_ASM` vs a C definition) before saying it is unmatched or proposing "once X is matched" as a next step. Same for open questions in notes: check whether matched C already answers them.

**Why:** 2026-10-25 I listed `BattleQueue1Execute` as asm to match, based on `BattleActionType14`'s comment ("still nonmatching"); the user pointed out it was already matched. The same session, BattleSetupFlags.md's "0x10 meaning open" was already answered by matched `batres.c` (`CommitBattleResults`).

**How to apply:** treat status words in comments/notes ("nonmatching", "open", "unpaired") as leads; confirm in the tree before repeating them. When an outdated comment caused it, add it to MiscCleanupNotes. Related: [[check-git-state-before-claiming]], [[check-notes-before-confidence]].
