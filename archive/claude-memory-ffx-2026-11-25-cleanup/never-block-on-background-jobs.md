---
name: never-block-on-background-jobs
description: run permuter and other long jobs in the background and keep talking; never wait on them in a foreground call
metadata:
  node_type: memory
  type: feedback
  originSessionId: d8b8ba71-d5b2-44ee-a8a4-83ea356c0bf1
  modified: 2026-10-31T01:09:13.723Z
---

Never block the conversation waiting for a long job (permuter, big scans, builds that take minutes). Start it in the background, report status right away, and check results when the notification arrives or the user asks.

**Why:** 2026-10-30, func_00255158: I ran a foreground `until ! pgrep permuter …; sleep` loop with a 10-minute timeout while the user was back; they said "Never block the discussion with a task that can be run in the background, like the permuter".

**How to apply:** permuter runs always `run_in_background` (or nohup) plus a Monitor/background waiter at most; no foreground sleep/poll loops. Give the user the current best and what's running, then end the turn. Related: [[decomp-reference-techniques]].
