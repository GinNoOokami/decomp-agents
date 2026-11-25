---
name: check-notes-before-confidence
description: "Before stating a TU boundary or rating a name as \"lead\"/\"unconfirmed\", read the DecompNotes that already cover it"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

**FFX: run `~/dev/ffx-analysis/scripts/notescheck.py <every address and name in the result>` before reporting any finding or proposing a tool/analysis** (user, 2026-11-04, after repeated rediscoveries: "You keep repeating the same mistake"); in the reply separate "already in the notes (file)" from "new".

Before answering a TU-boundary question or rating how well a name is backed, grep the project's notes (FFX: `~/dev/DecompNotes/FFX`) for the symbol/field first and quote what is already recorded.

**Why:** on 2026-10-22 (BattleCmdScriptDispatch session) I claimed BATTLE_Main shares a TU with the dispatcher from rodata alone; BattleOverlayOverview.md already said rodata can't show that (same-parity objects leave no trace) and had the U1/U2/U3 split with a build test. The same day I called `singleToMultiAnimTable` an unconfirmed lead while BattleNameCandidates.md and MiscCleanupNotes.md already recorded `unk20` as an animation ID and `unkB8` as the target count from matched callers. The user had to correct both.

**How to apply:** FFX: TU and system questions → `SystemSplit.md`, archive/loader → `Archive.md`, toolchain → `Toolchain.md`; FF7 (for reference): TU questions → BattleOverlayOverview.md ("Summary: TU boundaries", "Candidate boundaries inside battle", "Rodata check") before reasoning from the binary. Name confidence → BattleNameCandidates.md, MiscCleanupNotes.md, BattleCurrentAction.md for the field. Related: [[cross-version-tu-analysis]], [[naming-high-confidence-only]], [[decomp-notes-docs]].

**Handoff summaries can garble notes (2026-10-22):** the session-8 handoff said `D_800A0108` "must stay at file scope (rodata order)"; ImportDataSegment.md actually said the opposite (`D_800A0108` could move, `g_BattleCmdDefaults` couldn't). Treat handoff claims as leads and check the notes (or test) before repeating them.

**Start from the notes' answer, then work only on what they leave open (2026-11-04, hddcopy.c):** my grep
showed SystemSplit.md "0x165E98–≈0x168B68 HDD install", yet I re-derived both edges from the layout db
and called 0x1661F8 "earlier than the notes". User: "What's the point in creating notes if you don't
actually look at them?" Before any boundary/region work: quote the notes' range and its strength in the
first reply, list only the open questions (here: are 0x165E98–0x166A90 the same file, exact end), and
spend tool work on those alone. When reporting, separate "notes already say" from "new".

**Same day, battle loads:** I promised "new facts" (allocation tags, destinations) from a tool
expansion; `builds/ps2/us_demo/alloccalls.py` already had every tag and Log.md Phase 4b had `func_001AD180`'s
subs/tags/fields. Before claiming a result or a tool idea is new: grep Log.md + all FFX notes for the
function addresses, and check Tools.md for a tool that already outputs it (run it).

**Before an RTL/gcc-source investigation, grep the case notes for the symptom (2026-11-07):** I ran
`rtl.py --diff` and read expr.c to explain a `%hi` hoisted above a call; Troubleshooting.md case 11
and the matching-lessons memory already had the mechanism (with func_0011D370 as the example).
Grep `FFX/Troubleshooting.md` and [[ffx-matching-skill]] for the asm sign first.
