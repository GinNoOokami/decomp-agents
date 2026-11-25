---
name: evidence-by-platform
description: Cross-build boundary evidence is grouped by platform/toolchain; only codebase-certain evidence (file names, source strings) is shared across platforms
metadata:
  type: feedback
---

FFX builds (jp_intl, US alpha, US demo, PC Steam, Switch later) share one codebase but each platform
has its own toolchain and output. Compare apples to apples: toolchain-dependent evidence (data order,
string pooling, codegen, hand-written objects, padding/deltas) is combined only within one platform.
Only evidence certain to come from the source itself (file names, `__FILE__`, other source strings)
goes into shared, cross-platform output.

**Why:** user, 2026-11-04, when planning multi-build cross-referencing (as done in FF7).

**How to apply:** `layout.save_evidence(..., scope=, build=, platform=)`; `combine.py --platform`.
Count results as distinct gaps: the same cut carried from three builds is one cut (I once reported
60 "exact cuts" that were 14 gaps).
Within PS2, alpha/demo using the same compiler as International is inferred (masked exact matches),
not verified: keep the build column and check before treating them as one toolchain. See
[[cross-version-analysis]], [[tools-reference]].
