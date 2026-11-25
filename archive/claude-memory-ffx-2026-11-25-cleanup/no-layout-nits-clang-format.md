---
name: no-layout-nits-clang-format
description: FFX - never flag brace/line layout (e.g. one-line function bodies) in cleanups; clang-format decides it
metadata:
  node_type: memory
  type: feedback
  originSessionId: c90454ab-162f-475d-a3c3-7efc470b931a
  modified: 2026-11-20T01:13:52.513Z
---

Never suggest layout changes (one-line vs multi-line bodies, brace placement, wrapping) in FFX cleanups.
`.clang-format` (BasedOnStyle LLVM, ColumnLimit 120; `make format`) puts short functions on one line
(LLVM default AllowShortFunctionsOnASingleLine: All), so one-liners are the formatter's output.

**Why:** I flagged one-line bodies twice in a session (2026-11-19); the user pointed out it's the
formatter's settings and no other agent raises it.
**How to apply:** check `.clang-format` before any formatting remark; layout is out of scope for
[[cleanup-checklist]]; related [[no-obvious-match-reports]].
