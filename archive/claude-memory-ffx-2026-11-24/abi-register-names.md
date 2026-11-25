---
name: abi-register-names
description: "Refer to MIPS registers by ABI names (s0, a1, v0, ra), as objdiff shows them, not splat's $16/$5"
metadata:
  type: feedback
---

When quoting asm or giving hints, use ABI register names (`v0`, `a0`–`a3`, `s0`–`s7`, `sp`, `ra`), not the
raw `$N` numbers in splat's `.s` files. Convert when quoting lines from the `.s` (`$16` → `s0`).

**Why:** 2026-10-30 the user: "I use objdiff, so can you use those register names" after a hint said
"follow `$16`", which they couldn't find in objdiff.

**How to apply:** every register mention in chat and notes; `$2`=v0, `$4`–`$7`=a0–a3, `$16`–`$23`=s0–s7,
`$29`=sp, `$31`=ra. Related: [[verify-hints-before-giving]].
