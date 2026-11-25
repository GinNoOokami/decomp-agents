---
name: explain-step-by-step
description: Explain arithmetic idioms (masks, folded rounding constants) step by step with one worked value; no dense derived-layout tables
metadata:
  type: feedback
---

When explaining code math (masking, round-up constants like `+ 0x607` = add + 0x3F, layout sums),
walk through it in plain steps with one concrete value (n = 1 → 0x1900), then say what uses the
result. Don't compress it into a table of derived sizes.

**Why:** 2026-11-04, func_00171A88: the user got "it's a size" from the first answer but couldn't
follow the table, the mask, or the unexplained 0x607; the step-by-step version worked.

**How to apply:** first answers on unfamiliar arithmetic: idiom name, steps, one example value.
