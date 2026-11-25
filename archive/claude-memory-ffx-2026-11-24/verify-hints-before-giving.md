---
name: verify-hints-before-giving
description: "Before giving decomp/matching hints, test them in a scratch compile; clearly mark any untested idea as a guess."
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

Before giving the user a matching hint, verify it. Compile the suggested change in the scratchpad (see [[scratch-match-workflow]]) and check that the asm moves toward the target, or at least prove the claim by reading the asm carefully. If a hint can't be verified, say so plainly at the start, for example "Untested guess:". Never present a guess as an explanation of the asm.

**Why:** on 2026-10-05, while the user was matching BattleAddAutoBattleActionByChance, I gave two confident hints that were wrong:
- "Swap the order" of `chance = 100` and the decrement. I misread the delay slot. The user's original order was right.
- "Use two locals" (a raw byte plus the chance). The real fix was the user's own `enabledCounter->materiaAttribute--` with an `s32` chance local instead of `u8`.

Both hints steered the user away from code that was nearly correct. The user said: "Verify the solution or prove your hypothesis to yourself before giving me the hints, or at least preface them with 'I have no idea what I'm doing, but have you tried this instead'".

**How to apply:**
- For every hint about instruction order, local types or expression form, check it against the target asm with a scratch-copy test before sending it.
- When reading asm order, account for branch delay slots.
- When a test shows several variants giving equal results, report the simplest one, the one closest to the user's code. Don't report a more convoluted variant first.
- Related: [[no-unrequested-decomp-answers]] still applies. Hints, not full solutions, unless asked.

**Citing sources:** never suggest citing "community docs" or a wiki in code comments or notes unless I've actually opened the page in this session. On 2026-10-06 I suggested "script 3 (death counter, per community docs)" from memory, and the user asked "which docs?". Say plainly when something comes from my recall, and offer to look up the real page.

**Reviewing "did I fix X?" (2026-10-08):** when the user asks whether they fixed the issues I listed, diff the whole function in scratch and report anything else structural too, not just the listed items. I confirmed 2 of 3 fixes on `BattleBattleActionQueueExecute` but missed an extra `if (arg0 > 0)` (duplicate `blez`) and an m2c `result = i ^ arg0` tail. They showed up a turn later, and the user objected: "I thought you said I fixed all the structural issues". Also, don't call an m2c artifact "optional/harmless" without compiling it. I'd said that about the `if (arg0 > 0)` guard, and it wasn't.

**Equivalent rewrites in explanations get used as code (2026-10-09):** to explain `temp_a3 = (temp_v1 ^ mask) & temp_v1`, I said it "simplifies to `temp_v1 & ~mask`". The user reasonably applied it, and it cost an instruction (`nor` instead of `xor`). Later I flagged it as their mistake, and they pointed out I'd given it. When I simplify an expression to explain it, say plainly "for reading only; keep the original form to match", or compile it first.

**Don't present emulator checks as "quick" (2026-10-09):** I suggested a "quick emulator check" that needed particular battle situations (cover changes, a hiding enemy). The user pointed out that means finding those situations in a 60+ hour game. Exhaust static checks first (asm scans for direct stores, pointer stores at the field offset, base-address loads, struct copies or clears), and say honestly what a runtime check would need to set up.

**"What's left" lists come from a line diff, not memory (2026-10-17, func_800A3ED0):** I told the user "my match only differs from yours at these five points" from memory; there were three more (u8 local, clamp placement, 0xFF vs -1), plus a `unit[i]` double-index bug I missed by reading only the asm diff. The user: "You said 5 things would lead to a match, and then dump 6 more on me". Before listing remaining fixes, `diff` the user's function text against my verified-matching copy, apply exactly the listed fixes to their file in scratch, confirm MATCH, and leave-one-out test each item so every listed fix is actually required.

**Hints from a tested version must reproduce it exactly (2026-11-07, func_00159EA8):** I matched in
scratch, then summarised it as "keep the second allocation's result in a local". The match needed
the chained `D_0058F9DC = temp_2 = call()`; the user's two-statement version was 136/149 words off.
Before sending hints derived from my match, apply the hints literally to the user's current file in
scratch and confirm MATCH; quote the exact statement form when form matters.

**variant.py MATCH is not a full match (2026-11-08, movie_start_debug_resulti):** it masks
relocations, so two stores with swapped symbols (`D_0058F9E8`/`D_0058F9EC`) still showed MATCH and
I called it a full match; the user's build failed the sha1. Before saying "matches", run the full
`make` (sha1 OK) in a scratch copy of the user's current tree.
