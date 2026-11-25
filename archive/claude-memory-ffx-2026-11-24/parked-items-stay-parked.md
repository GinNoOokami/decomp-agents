---
name: parked-items-stay-parked
description: Handoff "open" items are parked on purpose; at session start check nonmatchings/ and NON_MATCHING blocks, don't propose known traps, ask what to work on
metadata:
  type: feedback
---

When resuming from a handoff, don't lead with its open items as the plan. Read the target first:
an existing `#ifndef NON_MATCHING` block or `FFX/nonmatchings/<addr>.md` means it was already tried
(and may be a known trap). Offer fresh areas and let the user pick.

**Why:** user, 2026-11-05: "ReadQEntryRead is a trap, you've tried numerous times ... You obviously
didn't even read the file or you'd have seen there's a nonmatching C version already ... we left all
of those open items open for a reason." They were at 42% of the week's usage on day 2.

**How to apply:** at session start, grep the tree (`NON_MATCHING`, `INCLUDE_ASM`) and the
nonmatchings README for every function a handoff names before suggesting it. Related:
[[handoffs-not-in-notes]], [[verify-match-status-from-source]], [[token-economy]].
