---
name: cleanup-checklist
description: "What \"clean up\" means for a freshly matched function — the user's standing checklist (locals, readability, nesting, names, comments) and how to deliver it"
metadata:
  node_type: memory
  type: memory
  originSessionId: bfce77a3-d28f-47f2-bbb6-51d5e4f9d558
  modified: 2026-11-20T04:48:19.121Z
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

When the user asks for cleanup after matching a function (2026-10-13, func_800AD0FC, "can you remember these preferences after matching a function when I ask for cleanup?"), do all of these by default:

0. **FFX: Switch counterpart first** ([[switch-counterpart-every-function]]): names, field meanings and inlining come from it.
1. **Remove unnecessary variables**: m2c temps, unused/shadowed decls, single-use copies, aliases of globals. Test each removal ([[cleanup-eliminate-unneeded-locals]]); keep a local only if the match needs it.
2. **Improve readability** (2026-10-19 miss on BattleActionType09: I skipped TARGET_RANDOM / TARGET_MULTIPLE_DEFAULT|TARGET_TOGGLE_MULTIPLE for targetFlags 0x80/0xC and dismissed `< 0x21` → `<= CMD_ENEMY_ATTACK`; the user did them and called the cleanup weak): **grep include/game.h and battle*.h for an enum for EVERY magic number by field (targetFlags, cmdIndex, status, attackFlags…), and accept enum-relative bounds like `<= LAST_ENUM`;** also drop `!= 0` on plain truth tests, turn m2c's `x -= 1` / `x += 1` into `x--` / `x++` (user, 2026-10-22, `D_801517B8 -= 1` in BattleCmdScriptDispatch; I left it and they had to point it out),  drop redundant casts/parens, replace magic numbers with existing enums/macros (STATUS_*, NUM_PARTY, START_ENEMY…), remove m2c `/* extern */` lines and put prototypes in the proper header (in FF7: game.h / battle_private.h; in FFX: whatever headers the user sets up, e.g. `include/game.h`).
2a. **Spell constants in terms of what they mean** (user liked this, 2026-10-20, BattleProcessStatusChanges: `~0x6000` → `~(1 << 0x0D | 1 << 0x0E)` where 0x0D/0x0E are handler-table indices): when a mask or number is built from indices, bit positions, sizes or other named things, propose the expression that shows its intent, and test it matches. The user wants these suggested routinely.
2b. **Split overlong lines with locals** (user, 2026-10-19, func_800A2DB0): when a line is long enough that clang-format wraps it (e.g. `enemy[formation[target - 4].enemyID].morphItem`), try introducing a local for a sub-expression (index or struct pointer) and test it matches. This is the one exception to "cut locals": a local that only exists for readability is fine when it matches.
2c. **Put switch cases in natural order** (ascending values / enum order), including inner switches. Missed on 2026-10-23 (BattleApplyActionEffect case 1's inner switch was SETUP, ACTION_END, LANDED_HIT, AFTER_DAMAGE; the user caught it). Case order sets code layout, so test the reorder with a full build, don't assume it's free.
3. **Reduce nesting**: guard clauses / early returns where the match allows; no goto.
4b. **FFX: return type of wrappers from the callers** (2026-11-19, TkCdReadSync / TkCdSize): a
   wrapper whose last act is a call leaves v0 set either way, so `void` and `return f();` compile
   the same. Grep the callers (`jal FUNC` then a use of `$2`) and make it return when they use it.
   **Exception (2026-11-19, func_001244B8 / TksceGsSyncPath):** ee-gcc 2.9 only emits a bare `j callee`
   tail jump when the call is a plain statement; `return f();` keeps a frame (jal, jr). So a target that
   is just `j f` must be `void w(void) { f(); }` or `T w(void) { f(); }` (falls off the end, v0 left
   from f); a framed jal+jr wrapper is `return f();`. I told the user `return` would match: it didn't.
4a. **FFX: the function's own rename is part of cleanup** (user, 2026-11-19, TkCdReadWithMallocWithMessage: "I thought the rename for this function was part of the cleanup"): when an agreed or evidence-backed name exists (e.g. a Switch name from step 0), "go ahead with cleanup" includes symbol_addrs.txt + C rename + make ([[add-symbol-names-for-user]]); don't ask separately. **Missed again 2026-11-19 (TkInitScene):** listed only prototype fixes, user asked "Why not suggest the rename before or during the cleanup as we have been doing?". Every cleanup proposal ends with the rename line when a name exists.
4. **Rename locals and parameters** (missed 2026-11-19 on TksceGsSyncPath: kept arg0/arg1 although the wrapped SDK prototype names them `mode`/`timeout`; a pass-through wrapper takes its callee's parameter names) to short, meaningful names in the project's vocabulary ([[naming-preferences]], [[naming-high-confidence-only]]).
5. **Comment non-obvious parts**: formulas, why a check exists, what data means. Comments must add information the code doesn't show ([[project-style-conventions]]). **Don't comment matching-required shapes** (e.g. "returning 0xFFFF directly doesn't match", a temp shared across branches): the user said on 2026-10-18 that once matched C is merged, nobody can change it to a non-matching shape without breaking the build, so such comments add nothing. Still mention match-required locals/shapes to the user in the review itself.

**How to apply:** this is still a proposal, so don't touch the tree unless asked ([[no-code-changes-without-permission]]). Build the cleaned version in scratch and check it there (FFX: `make` sha1 ✓, `make format`, per-function diff). Then show the user the code plus the header changes, list which locals were kept because the match needs them, and say which names or comments are guesses.
- **Bug comments only when the consequence isn't visible** (user, 2026-11-18, TkCloseControler: negative port falls through to `D_0078A7C0[-1]` → a `// bug:` comment is worthwhile because the fall-through's effect isn't obvious). Otherwise:
- **No comments on what the code visibly does, even latent bugs** (user, 2026-11-14, func_002D9028: "Comments like 'Reuses the table index' are not useful"). A reused variable or its consequence is visible in the code; bug write-ups go in the notes (TkPlatformLibrary.md had it). Keep comments to intent/meaning the code can't show (e.g. why disc names are upper-cased).
- **Initialize at the declaration** (user, 2026-11-25, MsSetBattleEncount): `u8 x = 0;` rather than
  `u8 x;` + `x = 0;` whenever the match allows; don't carry over m2c's split form in proposals.
