---
name: suspect-merged-structs-first
description: "When asm reaches one global's fields off another global's base register, raise \"these are one struct\" before reaching for casts"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

If the target addresses fields of global B at a fixed offset from global A's base register (e.g. `lhu 0x220C($s4)` with `$s4 = &g_BattleWork`, landing in `g_BattleSceneContext`), say first that A and B are probably one object in the original (A extended, or a parent aggregate), and check adjacency (A's address + size == B's address). Test that in scratch before any cast-the-base trick like `((B*)(&A + 1))->field`.

**Why:** On 2026-10-11 (func_800A38FC) I jumped to a `(BattleSceneContext*)(work + 1)` cast and presented it as what the match "needs". The user pointed out that a merged struct should have been my first suspicion, especially after the earlier BattleData refactor (one 0x178 struct replacing scattered globals, see `~/dev/DecompNotes/FF7/BattleSceneDataTheory.md`). The aggregate version matched naturally and kept every checksum and psyz green.

**How to apply:** GCC can't produce cross-symbol offsets from separate externs, so any such access is evidence of a single object. Lead with the struct-layout hypothesis and the adjacency check. Offer casts only as a stopgap, labelled as a hack, and never say a hack is "needed". See [[question-existing-struct-boundaries]] and [[matching-lessons]].
