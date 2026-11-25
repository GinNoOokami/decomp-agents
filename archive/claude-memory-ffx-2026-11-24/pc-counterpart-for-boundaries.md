---
name: pc-counterpart-for-boundaries
description: FFX - while matching, look up the PC counterpart too, not just Switch; MSVC PC keeps helper boundaries (static inline) that Switch inlines and unrolls
metadata:
  type: feedback
---

When matching a function, find its PC counterpart (`builds/pc/out/disasm.txt` / `decomp.c`) as well as
the Switch one. Do this especially when the Switch body is unrolled or bigger than PS2, or when the
match is stuck on one codegen detail. The PC build keeps function boundaries that the Switch build
inlines and unrolls. A helper visible on PC gives the PS2 inline's real signature, so I don't have to invent one.

**Why:** 2026-11-22, func_002D2890 (pad auto-repeat). In the matching session (2813f543) I only used Switch
`FUN_0058be40`, which had the loop unrolled. I never opened the PC exe. PC `FUN_00889940` is a 4-iteration
wrapper that calls the 6-argument helper `FUN_00889980(ptr, arg1, group, curr, prev, set)`, which masks its
by-value `curr`/`prev` in place. A `static inline` with that shape matched exactly, and another agent
found it from PC. My own inline tries used invented boundaries (timer by pointer, fresh masked
locals) and got 16-35 words. I read [[ffx-pc-port-workspace]] as "only for goto soup", and the
Switch step ([[switch-counterpart-every-function]]) made me think counterpart lookup was done.

**How to apply:** For each function, report both counterparts. If the Switch body is unrolled
or inlined, read the PC body for the call structure before trying inline experiments. Find it through a known
PC caller's callees (here `TkScanControler`'s counterpart calls 0x889940 twice). Related: [[switch-inlining-trap]].
