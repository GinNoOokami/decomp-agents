---
name: repo-tools-high-quality
description: "FFX - tools living in ~/dev/ffx/tools must be high quality, readable, comprehensible to a human; analysis-folder scripts can be rougher"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 12226579-bf6a-4f10-bb08-326e32876dc3
  modified: 2026-11-23T07:57:58.531Z
---

Rule (user, 2026-11-22): any tool that lives in the repo (`~/dev/ffx/tools/`) must be high quality,
readable and comprehensible to a human. Scripts in `~/dev/ffx-analysis/scripts` don't have to meet that
bar, but moving one into the repo (planned for the "daily drivers", not yet) means bringing it up to it.

**Why:** the repo is the deliverable; its tools get read and maintained by people, not just run.

**How to apply:** for a repo tool, write it clean from the start rather than promoting a patched-up
prototype: small single-purpose functions, named and commented regexes, data types (dataclasses) over
tuples, docstrings that explain the approach, no clever one-liners; black-formatted. Prototype in
scratch if useful, then rewrite before proposing it for the tree. Relates to [[tools-lean-evaluate-first]],
[[repo-self-contained]].
