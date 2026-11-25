---
name: score-is-not-structure
description: A lower objdiff % doesn't make a variant wrong; judge each piece by whether its local shape matches the target, since several right pieces can each cost % until combined
metadata:
  type: feedback
---

Don't drop a variant because the overall match % went down. A change can fix one region (e.g. the poll loop's compare order now matches the target) while costing elsewhere, and the right shape may only show once several such pieces are combined (user, 2026-10-31, func_001632B8; same lesson from FF7).

**Why:** % is a sum over the whole function; greedy hill-climbing on it discards correct pieces and steers toward unnatural code.

**How to apply:** for each variant, check the region it targets against the target asm (branch order, block placement, registers there) and report that, not just the total. Keep pieces that are locally right even when the total drops, and say which mismatch they trade against. Related: [[no-fake-matches]], [[logic-first-then-match]], [[gcc-sources-before-rewrites]].

**The other direction too (user, 2026-11-04, the maze):** a *better* score isn't progress either when
it was reached on an implausible path. Score = distance to the exit, not distance along a path that
gets there; a dead end 5 m from the exit still needs backtracking. I called ReadQEntryRead's
163 -> 112 differing words "progress" while that version rested on gotos neither of us thinks are
original. Report progress as: which paths are ruled out, which target details are now explained,
and whether the current shape is plausible source. Never present a score gain on a diagnostic
(goto/steering) version as getting closer to a match; say it's a diagnostic.

**And don't overrule measurements with a mechanism (2026-11-05, func_0015B490):** I called an
if/else + `return result` version "structurally closer" (it got loop.c to process the loop) while it
was worse on every measure (175 vs 95 differ, struct 88.9 vs 94.6, wrong frame); the user asked "Are
you sure that's right?". Struct %, frame size and spills are evidence too: when they disagree with
my theory, the theory is what needs checking, and the version gets reported as a diagnostic.
