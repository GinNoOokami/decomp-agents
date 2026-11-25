---
name: explain-game-context
description: "When explaining what a flag/field/data block does, also say where in the game it's used (which battles, which feature), not only how the code uses it"
metadata:
  type: feedback
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

When I describe a mechanism found in code (a flag, a field, shared memory), the user also wants the in-game context: which scenes or battles trigger it and what player-facing feature it serves. Find it from data (SCENE.BIN writers, formation/enemy names) and corroborate with community sources, labelled by strength.

**Why:** 2026-10-21, the "arena" HP-carry block: I explained how BattleInitMain and BattleResultsMain use the memory, and the user said I hadn't described "where in the game (which battles, what feature or context)". Finding the scene writers showed it was the Bizarro∙Sephiroth multi-party fight, not Battle Square, which overturned the field name and my struct comment.

**How to apply:** for an unexplained bit or field, find its writers in the data (FF7 used SCENE.BIN scans; FFX: the archive files, see `Archive.md`), then corroborate with community sources ([[community-tools-corroborate-only]]). Don't carry an existing name's story ("arena") into my names or comments until the data backs it ([[naming-high-confidence-only]]).
