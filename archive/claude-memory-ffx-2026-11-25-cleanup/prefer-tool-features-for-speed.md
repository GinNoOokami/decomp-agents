---
name: prefer-tool-features-for-speed
description: Prioritize workflow speedups that use built-in splat/m2c/objdiff features (symbol sizes, ignore, types, options) so m2c output needs less manual cleanup
metadata:
  type: feedback
---

Prioritize workflow speed improvements, and get them from the tools' built-in features first (splat `symbol_addrs` attributes like `size:` / `ignore:true` / types, m2c options and context, objdiff settings) before writing our own scripts (user, 2026-10-31, after sizing `D_0078DF98` made m2c emit `D_0078DF98.unk2C[arg0]` directly).

**Why:** "the less I have to stare at mangled m2c output, the quicker we can progress."

**How to apply:** when a function's m2c output needs hand-fixing that a symbol size/type or tool option would prevent, suggest that fix (tested in scratch) rather than just fixing the C. Keep a lookout for repeated manual cleanups and propose the tool feature that removes them. Related: [[ffx-repo-conventions]], [[splat-c-file-gotchas]].
