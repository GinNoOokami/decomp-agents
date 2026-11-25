---
name: globals-types-are-guesses
description: FFX - extern declarations of D_ globals in the user's TUs are guesses; derive shape from target offsets/strides, never hold them fixed
metadata:
  type: feedback
---

Every `extern` type/shape for an unnamed global (scalar vs array, element type, dimensions, struct
type) in the user's C files is a guess, not established fact. Derive shapes from the target (field
offsets, strides, access width) and say which declaration a hint depends on.

**Why:** user, 2026-11-10 (func_002D1EA0): "Make no assumptions about the current types of globals;
they are all guesses." The match needed `u8 D_00789720[0x1000]` changed to a 2-D `[1][0x1000]`
(stack pointer stepped by 0x1000); holding the existing declaration fixed had blocked earlier attempts.

**How to apply:** when a variant fails, include declaration shape among the levers; when proposing
a declaration change, cite the asm evidence (stride, offsets) and label it inferred. Related:
[[distrust-unnatural-code]], [[ffx-matching-lessons]].
