---
name: cleanup-applies-trivial
description: "FFX - cleanup trivial items and agreed header/extern moves apply directly when git diff is clean (scratch build first), then report; judgement items still proposed"
metadata:
  node_type: memory
  type: feedback
  originSessionId: c90454ab-162f-475d-a3c3-7efc470b931a
  modified: 2026-11-21T23:44:11.642Z
---

Working proposal (user, 2026-11-20): on "cleanup for X", apply the trivial items myself, then report:
Switch-backed renames (body + position checked) incl. symbol_addrs, parameter renames from the callee/
Switch, moving m2c `/* extern */` lines to their header/extern block, offset tags, side-by-side
`// gTkName?` comments on settled globals. Still propose and wait for: types, our own names, lead-
comment names, constants/defines, struct changes, anything uncertain.

Condition: only when `git diff --quiet` (no unstaged changes; staged WIP is fine). The user stages
WIP before conferring but sometimes forgets or stages right after prompting, so check immediately
before editing, not at the start of the turn; if unstaged changes exist, propose as before.
Run `git diff --quiet; echo $?` itself; don't eyeball `git status --short` (2026-11-20: I read
`M  file` = staged-only as unstaged twice, and proposed instead of applying; again 2026-11-23 for
several battle cleanups in a row: the user called it out). First command of every cleanup turn:
`git diff --quiet; echo $?`.
Missed again 2026-11-24 three times in one session (MsGetFileSize, MsWalkAbilityProcess,
MsSetCountryName cleanups: all staged-only, all proposed; user: "you're doing the back and forth
thing again"). Enum-for-literal swaps (GAME_REGION_*) tested in variant.py count as trivial too.

Extended (user, 2026-11-21): agreed header/extern moves (declarations between headers, new area
headers, includes, checked lead comments) also apply directly under the same condition: scratch
build, apply, build, report the diff. Header names and scope are still asked first.

**Why:** retro 2026-11-20 ([[session-mini-retro]]): nearly every cleanup was proposal -> "go ahead".
Overrides the per-function apply rule ([[apply-means-edit]]) only for the trivial list above.
