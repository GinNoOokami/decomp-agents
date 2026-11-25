---
name: naming-preferences
description: "The user's general naming preferences (verbs, one vocabulary per system, avoiding overloaded words, local consts, self-contained comments); FF7-only vocabulary left out"
metadata:
  node_type: memory
  type: memory
  originSessionId: 3a80d9cf-8c79-4fe8-a164-ccceb3de3d50
  modified: 2026-11-05T19:41:06.878Z
---

_Carried over from ff7-decomp on 2026-10-30, general parts only (FF7 vocabulary such as the `BattleUnit*` prefix and timer/sequence terms left out); examples are from FF7._

Learned during the battle-timer renames (2026-10-11):
- **Verbs:** dislikes `Recalc` (says nothing `Calc` doesn't). Rule of thumb from a survey of existing names: Get = returns a value; Set = derives/decides a value and stores it; Apply = applies an effect to something; Update = recurring per-frame step. Don't pick `Apply` for a function that only stores a rate.
- **One vocabulary per system, consistent across globals, fields, locals, macros, functions.** Settled timer vocabulary: Global/Var/Const timers (from the Battle Mechanics FAQ's V-/C-Timer), `varSpeed`/`constSpeed`, `…TimerAccum`, `…Frac`, ticks. The user rejected an "ElapsedTime" name for being outside that vocabulary, and Unity-style Scaled/Unscaled to avoid mixing.
- **Avoid overloaded words:** "Unit" means a battle unit here (`BattleUnit`, `…UnitMask`), so not for timer units; "db" reads as database (rejected `DbIndex`). Name constants for meaning, not mechanics, but not misleadingly (`_FRAC_BITS` over `_SHIFT`; `_UNIT 13` rejected as reading like 8192).
- **Macros named per quantity** read better than generic ones (`BATTLE_HP_DELTA(x)` over `BATTLE_TICKS(x, HP_BITS)`); the user dislikes "WHOLE".
- **Comments must be self-contained:** don't rely on the reader knowing the FAQ; a "see the FAQ" pointer is fine as an extra. No "thread" wording for interrupt vs main-loop code.
- **Resolve ambiguity in the change that introduces it (2026-10-17).** If a new name collides with an existing one, rename the existing one in the same change; don't defer it to a follow-up rename.
- Present 2-3 options with call-site examples and a recommendation; the user often picks or tweaks. Related: [[naming-high-confidence-only]], [[project-style-conventions]].
- **Recent comments aren't established convention (2026-10-13).** Before citing a comment or doc as existing terminology or as independent evidence, check whether it's new (`git log -L`/blame, recent commits); the user may have just written it. The user added `AttackData.damageCalcID`'s "hit and damage formula selectors" comment themselves; I first cited it as independent support for the nibble split, then took "selectors" as their chosen vocabulary. They meant neither: it's a recent note, so give it little weight and choose names on their merits.

**Local consts over global defines (2026-10-19):** when a named constant is used in only one function (e.g. a status mask), make it a `const s32 name = ...;` local at the top of that function, not a file-level `#define`. Only suggest a define or enum when several functions share the constant. The user's func_800AA950 `coverBlockStatuses` matched the same as the define did.

**FFX naming policy (user, 2026-10-31) — differs from FF7's:** a general or vague name is better than a hex name and can be refined later; don't wait for full evidence. Keep the address name only while we have no idea what a thing does or owns. Name from what the code shows (role, owner), state the basis in a comment when it's a guess. The FF7 high-confidence-only rule ([[naming-high-confidence-only]]) does not apply here.

**An explanation isn't a decision (2026-11-01):** the user explained why verb-last names like
`PcFileTblSet` might exist (Japanese is verb-final); I recorded it as an adopted rule and had to
revert. Only record a convention as decided when the user says so; otherwise note the reasoning
under the open question.

**Our own earlier names are not evidence (2026-11-01):** I justified `ExFile` by "matches the
`exFile` locals", but an earlier session of mine had named that local. Before citing a name in the
tree as support, check who introduced it (git log -S); if it was us, trace it to the real source
(here the `"pfs0:/ex/%s"` string) and say so.

**But don't dismiss earlier names either (user, 2026-11-05):** names from our joint cleanup passes
are best guesses from what we knew then, not "no evidence". I called `last_read_time` "no evidence,
came in with a cleanup pass". Frame it as: named in an earlier pass from X; new fact Y changes the
meaning, so rename to Z. Revisit names whenever new context arrives.

**Common words over jargon (2026-11-03):** for a field that tags which variant a record is (read vs seek request), the user prefers `type` (`ReadQEntry.type`, `ReadQType`) over `kind`; "kind" is compiler-codebase jargon they rarely see. Still avoid `mode` when the TU already uses it for other things. Function names spell words out: `Debug…`, not `Dbg…` (globals keep the existing `dbg_` prefix).

**Vocabulary changes cover everything (user, 2026-11-15, 2D19F8.c):** when names switch to a new
vocabulary (e.g. our `VBlankThread` → Switch `SyncVCallback`), "consistent" means all of the
system's identifiers at once: functions, statics/globals, inline helpers, defines, header
entries, symbol_addrs. I renamed only the functions and asked about the rest; the user had to
repeat it.

**Area header names (2026-11-21):** name a header after the area's evidence-backed module prefix
(Switch/original names: `atel.h`, `sqs.h`) rather than a generic concept word; I proposed `event.h`
for the Atel block and the user asked why. Generic names (`game.h`, `battle.h`) only where no prefix
covers the group.

**Full words, no invented abbreviations** (user, 2026-11-24, FFX MsBtlListInit: I proposed `fieldOfs`/`encountOfs`;
"No need to make the code unnecessarily cryptic when the original symbols don't directly support it").
Abbreviate only where the original names do (Switch `Btl`, `Ms`, `Num`, `CDpos`); otherwise
`fieldOffset`, not `fieldOfs`.
