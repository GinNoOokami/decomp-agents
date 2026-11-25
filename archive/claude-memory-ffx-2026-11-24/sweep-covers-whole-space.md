---
name: sweep-covers-whole-space
description: Sweeps must cover the whole search space or say exactly what they fixed; never conclude "not X" from a partial sweep
metadata:
  type: feedback
---

When a sweep (vsweep orders, type packs) fixes part of the space, such as the first statement or a sampled subset, the conclusion is limited to that part. State that limit, and never call the problem "structural" or "not statement order" from it.

**Why:** 2026-11-09, `func_0011F8F8`: my `vsweep orders` used `--from` on the second line, so the first statement stayed fixed, and it sampled 4000 of 9! orders. I then told the user that statement order wasn't the missing piece. The real fix was an order that moved a different statement first. The user called it a big miss.

**How to apply:** include every statement in an order sweep (all lines, or a moves sweep over each line first), report the coverage as "N of M orders, X fixed", and label any conclusion as limited to that coverage. Related: [[community-tools-corroborate-only]], [[score-is-not-structure]], [[usage-budget]].
