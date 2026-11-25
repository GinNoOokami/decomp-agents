---
name: copy-target-asm-at-start
description: "as soon as the user starts a new function, copy its target .s into the scratchpad before they build"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

When the user starts working on a new function, copy its target `.s` (FFX: `asm/jp_intl/nonmatchings/<unit>/<func>.s`) into the scratchpad right away, before they run m2c/decompile and build.

**Why:** their first build after replacing INCLUDE_ASM deletes the target .s from their tree. On 2026-10-15 (BattleActionType16) I needed the target for matching tips and had to rebuild a whole scratch repo to regenerate it. The user asked me to do this going forward.

**How to apply:** at the first mention of a new function (or when it shows up in their diff), `cp` its .s to `$S/target/`. FFX: splat writes `.s` only for functions still `INCLUDE_ASM`, so the target goes missing after the next re-split (any yaml/`symbol_addrs.txt` change, `make distclean`), not on a plain build (inferred). Related: [[check-diff-for-current-function]].

**Scratch only (2026-11-22):** never commit target `.s` into DecompNotes nonmatchings folders. I did for 002D2890 and the user had it removed; note folders hold best-version C only.
