# Lessons from past splits

Each with the split that taught it. Rules that grew out of a lesson live in SKILL.md; this file
keeps the cases.

- **Static `.bss` beats reference edges** (1B8920, 2026-11-26): `D_00662080` used by 0x1B8C30 and
  0x1BBCC0 rejected two khasinski edges inside that span.
- **`.rodata` skip gives a window, not a point** (1B8920): the skip only says the new file's first
  `.rodata` user is after the old file's last; functions in between with no data can go either
  way. Choose by goal and say so.
- **No pad by default** (1B8920/1BC308): an INCLUDE_ASM jump table's `.s` carries its trailing zero
  words, so the C object's `.rodata` already ends 16-aligned. Build without a `pad` first; add one
  only when the build shifts (the 1AF4E8 case: matching the last function drops that padding).
- **Read the checklist before the pilot**, not after (1B8920: four checks were run only after the
  proposal; none changed it, but the order matters when one does).
- **Goal beyond the own-signal end** (1CCAF8): the file's statics stop at 0x1CE9F0, but matched
  `1ACEA8` calls 0x1CEB60–0x1CED10. The end was extended to 0x1CED80 on a continuous Switch run
  (`MsEffectForceFree`, `MsEtEffectInit` … `MsEtEffectFree`), labelled a lead, with the move cost
  stated. khasinski's tu_196 end and the alpha boundary hint 0x1CEC20|0x1CECE0 were both rejected
  because static `.bss` (0x662120, 0x662124) crosses them.
- **Top callee outside the split** (1CCAF8): `func_001CCA48` (24 callers) sits before the chosen
  start window, in a file with no start edge; the effect block next to it was split and the
  `MsStructClear?` file named as the follow-up.
