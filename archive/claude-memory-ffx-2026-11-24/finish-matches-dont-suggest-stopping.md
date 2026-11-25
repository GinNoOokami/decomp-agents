---
name: finish-matches-dont-suggest-stopping
description: Don't call it quits on a close match on my own, but make the cost visible and offer parking (nonmatching entry) as an option once a match turns stubborn
metadata:
  type: feedback
---

From one incident, not a standing order: on 2026-10-11 (FF7, BattleRecalcUnitSpeed, 116/115 insns,
one register-sharing issue) I said "I'd stop here"; the user replied "Nah, we're in too deep now.
Let's finish this", and it matched minutes later. So don't unilaterally recommend stopping when the
next step is concrete and cheap.

That is not "never mention stopping". The user's budget is limited (see [[usage-budget]]): on
2026-11-05 they parked two stubborn control-flow cases themselves (DiscCheck's gotos: "not worth
it ... much lower hanging fruit"; func_0015B490: "Let's move on, I'm tired of this one").

**How to apply:** when a match is stuck, give the next concrete step with a rough cost (cheap probe
vs. RTL deep dive), and once it's been stubborn for a few rounds, list "park it with a
`NON_MATCHING` + nonmatchings note entry" as one option next to the next step. The user decides.
When they park it, write the entry and move on; don't re-propose it later ([[parked-items-stay-parked]]).
Ways forward are RTL dumps, gcc source and small probes ([[gcc-sources-before-rewrites]]), with the permuter only as a
last-ditch step once structure is settled ([[permuter-low-value]]).

Parking mechanics: comment + `#ifndef NON_MATCHING` / `INCLUDE_ASM` / `#else` C `#endif` (pattern in `src/lib/moviefunc_ac.c`); if C callers in the same TU have no prototype, add one above the `#ifndef` (INCLUDE_ASM declares nothing). Note via `nmnote.py` + `--facts`, tried variants via `variant.py --log`.

**Before offering parking (2026-11-22):** read both counterparts first, PC disassembly and Switch,
plus the refs ([[check-refs-first]]). func_002D2890 was parked after 30+ variants and RTL traces
without one look at PC, which showed the inline helper that matched. No fixed order; every source
covered before parking ([[gcc-sources-before-rewrites]]). See [[pc-counterpart-for-boundaries]].
