---
name: ffx-single-version
description: "~/dev/ffx builds only jp_intl; never suggest checking/updating an \"alpha build\" or alpha symbol_addrs"
metadata:
  node_type: memory
  type: project
  originSessionId: a69d1169-1fcc-4ea3-97c9-ca7227bb20b9
  modified: 2026-11-03T02:16:53.434Z
---

`~/dev/ffx` has exactly one build version: `jp_intl` (`config/jp_intl/` only). There is no alpha build in the repo, even though git history has a "Build alpha again" commit and old handoffs mention an alpha `symbol_addrs.txt`.

**Why:** user corrected this on 2026-11-02 ("there is only one version right now... it's the second time you've mentioned it"). I had told them to "run make for the alpha build too" after a flag change, and a handoff carried a stale "alpha symbol_addrs doesn't have read_q_req_no" item.

**How to apply:** verification = one `make` (jp_intl). Don't list alpha follow-ups; drop such items from handoffs. If versions are added later, `ls config/` will show it. Alpha/US-demo binaries still exist as analysis inputs in ffx-analysis ([[ffx-start-here]]), which is separate from the repo's build.
