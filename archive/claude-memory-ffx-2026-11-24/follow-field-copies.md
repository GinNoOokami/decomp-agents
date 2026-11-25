---
name: follow-field-copies
description: "Before saying a field/bit has 'no reader' or queuing a debugger check, follow every copy of the value into other tables and try a static answer first"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

A "no engine reader" claim must cover copies of the value, not just the field itself. And a handoff "debugger check" item should get a static pass (matched C + SCENE.BIN/KERNEL.BIN data) before the user sets up a fight.

**Why:** 2026-10-25, stateFlags 0x01/0x04 had been called "no reader on PSX or PC" for weeks, because sweeps only covered `stateFlags` and `formation[].flags`. The real reader used a third copy: `func_800B3FFC` copies the formation flags into `D_800F7DF4.enemies[].flags`, read by `BattleInitModelsAnimAndColor`. A read breakpoint couldn't find it either: the copy happens once at battle start, and `BattleUpdateUnitMasks` reads `stateFlags` twice a frame. In the same session, 5 handoff debugger items (turnFlags 0x04, stateFlags 0x800, cycleFlags 0x04/0x08, setupFlags 0x02/0x08) were settled from code and data with no fight needed.

**How to apply:** for each "no reader" bit, grep for every store of the field's value (`lw` of the field followed by an `sw` elsewhere) and repeat the reader scan on each destination. For each debugger item, check the writers and readers in C first; give a breakpoint plan only for what's still open. Prefer pokes (edit the stored value, or poke the consumer's own flag) over read breakpoints on fields that are read every frame. Related: [[debugger-only-when-c-weak]], [[pcsx-redux-debugging]], [[state-evidence-at-real-strength]].

**"No caller" claims must include tail-jump wrappers (FFX, 2026-11-05):** the region setter 0x2D9430 was noted as having "no `jal` caller", but an 8-byte wrapper 0x2D9458 (`j 0x2D9430`) had one, a script handler. Check `refs` of kind `jump`/`word` as well as `call`, then repeat on each wrapper.
