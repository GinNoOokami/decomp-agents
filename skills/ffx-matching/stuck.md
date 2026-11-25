# Stubborn matches and parking

## Judge by structure, not score

- A variant that fixes its target region but lowers the total is kept; say what it trades against.
  Never drop a lever for scoring worse alone: levers interact (Sg_ScreenshotWrite's second fix scored
  21 words alone and was needed).
- A better score on an implausible path (gotos, steering) is a dead end near the exit, not progress
  (ReadQEntryRead: 163 -> 112 differing words, on gotos neither of us thought original).
- Measurements beat mechanisms: when struct %, frame size or spills disagree with a theory, the theory
  needs checking and the version is a diagnostic (func_0015B490: a version called "structurally
  closer" was 175 vs 95 differing, struct 88.9 vs 94.6, wrong frame).

## Cover every idea source before parking

No fixed order; all else equal, cheaper first. Before offering parking, each of these has been looked
at, or is named in the report as skipped:

- PC and Switch counterparts (PC for helper boundaries the Switch build inlines: func_002D2890 matched
  from PC's 6-argument helper after 30+ variants never opened it).
- The ref repos and matched siblings.
- These lesson files and Troubleshooting.md.
- gcc source and RTL traces, each turned into concrete source shapes and tested
  (Sg_ScreenshotWrite: a trace and a permuter hint were never turned into the obvious test).
- The stuck statement's natural positions and the declarations of globals involved.
- Compiler flags, as a whole-build hypothesis, when a trace points at an option-controlled pass.
- Permuter hints, last.

## Tried tables

- Every tried table gets a "held constant" line: the axes every row shares (where a value is
  narrowed, locals vs parameters, statement placement). Two parked functions in a row had a missing
  axis.
- Results from invented signatures or shapes are "N invented shapes tried, not ruled out" unless a
  counterpart backs the shape.
- A diagnostic that matches with an empty asm barrier leads to ordinary C that hides the same value
  from gcc (an in-place `x &= m`, a reassigned by-value parameter, an inline's parameter copy). Test
  those before parking.
- Sweeps state their coverage; a partial sweep never proves "not order" or "structural".

## Cost and parking

- Give the next concrete step with a rough cost (cheap probe vs RTL deep dive). Being stuck means
  switching to the source, never recommending INCLUDE_ASM. Don't recommend stopping on your own when
  the next step is concrete and cheap (an FF7 "I'd stop here" at 116/115 insns matched minutes
  later); once a match has been stubborn for a few rounds, list parking as one option beside the next
  step. The user decides, and parks things themselves when the budget says so.
- **Parking** (when the user says so): wrap the user's *current* C (re-read it first) in
  `#ifndef NON_MATCHING` / `INCLUDE_ASM` / `#else` / C / `#endif`, comment starting with the substance
  (words off + cause); add a prototype above if same-TU C callers need one. Write
  `FFX/nonmatchings/<ADDR>.md` + README row with status `open` (via `nmnote.py --facts`; tried variants
  via `variant.py --log`), commit and push the notes, full build. Best-version C only in the note
  folder, never target `.s`.
- Parked items stay parked; don't re-propose them.
