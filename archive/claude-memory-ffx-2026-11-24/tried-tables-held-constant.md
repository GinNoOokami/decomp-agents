---
name: tried-tables-held-constant
description: FFX nonmatchings - tried tables must list the axes held constant; invented-shape failures don't refute a lead; a barrier diagnostic means "find the natural construct", not park
metadata:
  type: feedback
---

When I write or extend a tried table in a nonmatchings note, add a "held constant" line: the axes
every row shares (where a value is narrowed, locals vs parameters, statement placement). Lead
results tested only with signatures/shapes I invented are recorded as "N invented shapes tried, not
ruled out" unless a counterpart (PC/Switch/alpha) backs the shape. A diagnostic that matches with an
empty asm barrier should lead to a list of ordinary C that hides the same value from gcc (an in-place
`x &= m`, a reassigned by-value parameter, an inline's parameter copy). Test those before offering parking.

**Why:** 2026-11-22, func_002D2890. 30+ rows all narrowed `current` in its initializer. The user's
inline lead was written up as "tried … all 16" from made-up signatures. The barrier diagnostic was
left as an open guess. The match was an in-place mask of a by-value parameter in the PC helper's shape.

**How to apply:** check this before parking anything. Related: [[pc-counterpart-for-boundaries]],
[[finish-matches-dont-suggest-stopping]], [[sweep-covers-whole-space]], [[no-fake-matches]].

**Repeat, 2026-11-22, Sg_ScreenshotWrite (0x11F2B8, parked 2026-11-09 at 10 words):** every
`written +=` placement I tried stayed before the pixel loop; the match puts it at the row's end.
The other half of the fix (`src` at function scope) scored worse alone (21 words), so I dropped it. Levers
can interact: re-test rejected levers combined with each new placement, and never drop a lever
for scoring worse alone ([[score-is-not-structure]]). Two parked functions in a row had a missing axis.
