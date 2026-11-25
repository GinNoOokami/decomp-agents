---
name: cleanup-eliminate-unneeded-locals
description: "During post-match cleanup, test removing every local that isn't needed for the match, including local copies of globals like `act = g_CurrentAction`"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

During cleanup of a matched function, try to remove every unnecessary local, and suggest it only after testing that it still matches. Candidates:
- local aliases of globals (`Unk800A8D04* act = g_CurrentAction;`): try using the global directly;
- m2c temporaries and single-use copies;
- variables that only rename an expression.

Keep a local only when removing it breaks the match. When that happens, note that the local is a matching requirement, as with the two stack arguments in `func_800AE82C`, which had to be computed into locals before the call.

**Why:** in `func_800AE82C` (2026-10-10), the `act` local, kept from m2c and our edits, made it look as if the original authors mixed `act` and `g_CurrentAction`. Using `g_CurrentAction` everywhere also matched. The mix was an artifact. The user said: "in the future let's try and eliminate unnecessary locals during cleanup, including local global assignments like this."

**How to apply:** make it a standard item in every cleanup review. Run an mrun variant that removes each local (one variant per local, plus one with all of them removed together), and report which removals still match. Related: [[reviewer-prefers-early-returns]], [[project-style-conventions]].

**Readability exception (user, 2026-11-05):** don't cut a local whose removal nests calls three
deep (`TaskBuffInit(n, func_0011D370(TaskBuffSize(n), ...), 1)`); the user restored `s32 size`.
Same rule as cleanup-checklist 2b: a local that only exists for readability is fine.

**Readability comes first (user, 2026-11-16):** propose removing a local only when the result is at
least as readable. A named intermediate that splits a compound expression (`s32 index = (... + n) & 3;
return &...->entries[index];`) stays, even if removing it still matches. Folding it into one long
expression or swapping it for a different local is "objectively less readable"; don't offer it.
