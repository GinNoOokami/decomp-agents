---
name: usage-budget
description: User keeps bumping into the weekly usage limit (since starting FFX); prefer cheap steps, estimate cost before deep dives, surface it when a task gets expensive
metadata:
  type: user
---

The user's Claude usage is limited and has been a recurring constraint for several weeks of the FFX
project (2026-11-05: 42% of the week's usage on day 2; "budget is not unlimited and it's something
I've been bumping into the last several weeks since starting this project").

**How to apply:** pick the cheapest step that can answer the question (grep / one probe before RTL
dumps or variant batches); before a multi-round investigation, say roughly how deep it goes and let
the user choose; say so when a match or analysis has turned expensive rather than silently
continuing. Related: [[token-economy]], [[ffx-matching-skill]],
[[pilot-before-frameworks]].
