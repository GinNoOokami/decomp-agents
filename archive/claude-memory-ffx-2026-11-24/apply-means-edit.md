---
name: apply-means-edit
description: "Apply X" / "go ahead and apply" from the user = explicit permission to edit their code with X
metadata:
  type: feedback
---

When the user says "apply" (e.g. "Go ahead and apply both"), they mean edit their code with the
proposed changes, not only notes. On 2026-11-07 I added only the Troubleshooting entries for
"apply both and add the memory"; the user meant the four code changes too ("I did actually mean
apply the changes to my code").

**Why:** the no-edit rule needs explicit permission; "apply" is that permission for the changes
just proposed.

**How to apply:** treat "apply" as covering the code changes under discussion. If it could also
mean notes, do both, or ask in one line when the scope is unclear. Re-read the file first: the user
may already have made part of the change (they had added the alloc macros themselves).

**Permission is per function (2026-11-16):** "Next func_X" / "Cleanup for func_X?" asks for a
proposal, even in a run of near-identical getters where the last several were approved. Only an
explicit "go ahead" / "apply" / "you know the drill, add the name" covers edits, and only for that
function. I edited func_002D25E8 on "Next func_002D25E8" and was blocked.
