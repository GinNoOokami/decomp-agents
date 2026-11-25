---
name: state-evidence-at-real-strength
description: "Describe what the evidence literally shows (the instruction, the adjacency) before the inference (a memset, one struct); split certain / inferred / unverified"
metadata:
  node_type: memory
  type: feedback
  originSessionId: e93c531e-71aa-4e34-9fbf-0de6c317f0d9
  modified: 2026-10-26T03:04:16.690Z
---

When citing evidence, quote what was observed and label the inference separately. Say "one `rep stosd` of 0x253 dwords from 0x99c750", not "one memset of 0x94C"; say "the script can address these bytes", not "they're the shared variable bank".

**Why:** 2026-10-25 the user challenged two overstatements in a row: "How are you so certain this is all part of the same shared script var bank?" and "So there isn't a single memset with a ptr and size of 0x94C? You made it sound like there was". Both were inferences presented as facts.

**How to apply:** for layout/struct/naming claims, give three tiers (certain from code / inferred / unverified) and what would settle the inferred part. Matching builds only prove addresses, not how the source was written. **Format in chat (2026-10-30):** keep it natural; don't tag certain claims (no standalone "**Certain.**" sentences); only flag what is inferred, unverified or less confident. Tracked notes may keep explicit tags. Related: [[user-verifies-findings-before-pr]], [[verify-hints-before-giving]].
