---
name: own-my-output-in-reviews
description: "Before flagging something in 'the user's changes', check whether I wrote it; when applying my proposal to another branch, check the names it references exist there"
metadata:
  type: feedback
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

When reviewing the user's diff, separate what they changed from what I wrote earlier. Compare against my last applied version (scratch copy or the conversation) and only attribute their own edits to them. If something I wrote is wrong, say "my comment/code is wrong" and fix or offer to fix it.

**Why:** 2026-10-21 I flagged "the new comment names a function that doesn't exist on this branch" as an issue in the user's adjustments. I had written that comment, against their other branch where the function had already been renamed, and didn't check when applying it to a branch cut from main. The user: "that was your comment man, you can't make it and then complain to me it's wrong lol".

**How to apply:** when applying a change to a different branch than it was written for, grep that branch for every function/symbol name in my comments and code. In reviews, diff the user's tree against my last applied state to see their actual edits. Related: [[check-git-state-before-claiming]], [[user-comments-are-theirs]].

**Commit author ≠ code author (2026-11-05):** the user commits code I wrote, so `git log`/`-S` showing their commit doesn't make a placement theirs. Before saying "you put X there", check the handoff/notes or say I don't know who chose it. I called the `extern.h` task block the user's placement; I had written it the session before ("You did, earlier today in the last session").
