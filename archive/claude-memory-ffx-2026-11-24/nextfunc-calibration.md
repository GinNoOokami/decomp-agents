---
name: nextfunc-calibration
description: "How to calibrate nextfunc.py's cost weights with the user's after-match ratings; reference picking rules"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 754a8c17-1642-414a-905e-28f6fedc0e24
  modified: 2026-11-07T00:21:11.346Z
---

After the user matches a nextfunc.py pick, ask harder/easier/same vs one reference function and log it (with why) in ffx-analysis/data/nextfunc_ratings.tsv. Fit weights only after ~15-20 pairs.

Reference rules: a function matched recently (this session ideally; the user forgets older ones), not part of a twin family, not an m2c-direct match ("harder" is then a given, 2026-11-23) (check by hand: --matched rows don't show twin tags), closest cost. Twin pairs are logged but kept out of the fit (the first of a family carries all the work).

**Why:** a reference from a libmscd twin family matched long ago gave no usable rating (2026-11-06). The user declined a --ref option: keep the tool lean, pick by hand.

If m2c's output matched as-is, log verdict `m2c-direct` with no reference (user, 2026-11-06: nothing to compare when no effort was spent); these are gap-0 data for the m2c-gap idea in ToolingIdeas.md.

Exclude matches whose cost was toolchain/infrastructure work (MsGetMapCenter: assembler + helper, 2026-11-23). Shared-struct functions are noisy: unknown fields/pointee types of a shared struct dominate their difficulty, and they get easier once the struct is mapped (user, 2026-11-23); say so in the row's why.

**How to apply:** name the reference when the user starts a function, before they know how it went. Related: [[tools-reference]].
