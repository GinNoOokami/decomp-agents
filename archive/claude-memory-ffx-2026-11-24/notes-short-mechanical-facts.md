---
name: notes-short-mechanical-facts
description: "Note X" = one or two lines, not an evidence write-up; facts a tool can check (symbol overlaps, struct folds) don't belong in notes
metadata:
  type: feedback
---

When the user asks to "make a note" of a small fact (e.g. an extern overlapping a struct), write one or
two lines, not a ~150-word evidence section with sub-bullets and test status. Facts a check tool can
re-derive (overlapping globals, which D_ labels fold into a struct) shouldn't need notes at all.

**Why:** 2026-11-22, the BattleData.md "0x316F70 split labels" section; the user said such notes add
little value and asked for an overlap checker (like FF7's `make lint`) instead.

**How to apply:** keep requested notes minimal; prefer a tool over notekeeping for anything mechanical.
Relates to [[state-evidence-at-real-strength]], [[memory-scope]].
