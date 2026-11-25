---
name: m2c-order-is-not-source-order
description: m2c/target store order is the scheduler's output, not the source order; question it before scheduling/RTL/aliasing theories
metadata:
  type: feedback
---

The statement order in m2c output, and the store order in the target asm, show where the scheduler put the stores, not where the source had them. Treat "the source order is X" as an assumption to test first, with cheap order variants such as moving whole groups (all the int stores together, a copy block in field order), before scheduling, RTL-dump or aliasing explanations.

**Why:** 2026-11-09, `func_0011F8F8`: I argued "the target keeps all ten stores in source order, so the original compiler saw them as aliasing", but the source order I meant was m2c's, copied from the target. The matching source put `D_00307C60 = 1` first (its store is fifth in the asm) and the five stamp copies together in field order. The user: "We know the store order from m2c/target can be very misleading. We should carefully scrutinize this assumption before resorting to expensive scheduling checks, rtl dumps, etc."

**How to apply:** when a diff is only order and registers, first try natural source shapes: related assignments grouped, copies in field order, flag/state setters together, and sibling functions' shapes. Check the ref decomps (khasinski-ffx, rozelette-ffx) too. Only then go to rtl.py or permuters. Related: [[sweep-covers-whole-space]], [[community-tools-corroborate-only]], [[gcc-sources-before-rewrites]], [[ffx-matching-lessons]].

**Case, 2026-11-22, Sg_ScreenshotWrite:** the target had `written += 0x600` in each row loop's
header, before the pixel loop; the source has it after the loop (end of row), and gcc moved it up.
I parked the function at 10 words after placing the add only where the asm showed it. When a
statement's asm position is stuck, try the natural source positions (end of row, after the use)
first ([[tried-tables-held-constant]]).
