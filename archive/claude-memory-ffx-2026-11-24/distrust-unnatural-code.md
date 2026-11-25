---
name: distrust-unnatural-code
description: "Never propose code nobody would write (odd casts, magic derived constants, required weird locals); look for the natural source shape: types, sizeof, macros"
metadata:
  node_type: memory
  type: feedback
  originSessionId: efe8d60a-fbc1-4f79-84ff-a38bf4cca4fc
  modified: 2026-10-20T23:10:48.152Z
---

If the only C that reproduces the asm is something no programmer would write, treat that as a sign the types are wrong. Example: `(&atk->cameraSingleID)[(flags >> 3) & 1]`. Look for a natural alternative before proposing it. Most often adjacent same-typed fields are really an array (`u16 cameraIDs[2]`), or two structs are really one (see [[suspect-merged-structs-first]]).

**Why:** user, 2026-10-20 (BattleActionType0C): "No one writes code like this. Always be skeptical of unnatural code like this when it arises, and consider alternatives. In this case, it's likely these are not two separate fields but a u16[2]". The array version gave identical code.

**How to apply:** when a variant matches but reads unnaturally, don't present it as the answer. Test a type/layout change that makes it natural (array, union, merged struct). Present that one, and mention the other users of the fields that would change.

**Magic constants and "required" odd locals (FFX, 2026-11-05, func_00171A88):** the user's first
match was `size1 = n * 0x1310; size2 = n << 5; return (size1 + size2 + 0x607) & 0xFFFFFFC0;` and I
"cleaned" it by testing removals, concluding every odd part was load-bearing. The user: "There's no
way this looked like this in the original… 0x1310 was a sizeof, and the 0x607 had to be computed."
The natural source was `ALIGN(0x5C8 + n * sizeof(A) + n * sizeof(B), 64)`: `sizeof` makes the sum
unsigned (size_t), which is what gave the `lui/ori` mask and kept the terms apart. Lessons:
- Constants that look like sizes (0x1310, 0x20), sums with alignment folded in (0x607 = 0x5C8 + 0x3F)
  or masks (`& 0xFFFFFFC0`) are derived values: rebuild them from sizeof/macros before matching.
- When a cleanup "needs" strange locals, a split shift, or a literal mask form, suspect a missing
  type property first (signedness from sizeof/unsigned, pointer vs int), not a quirky author.
- Cleanup isn't done when every removal fails; ask what the original programmer would have typed.
