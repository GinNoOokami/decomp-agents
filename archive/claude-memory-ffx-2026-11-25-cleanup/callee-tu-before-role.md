---
name: callee-tu-before-role
description: Before describing what a branch or field is for, check which TU its callees live in (an fd read was the HDD install, not a host file)
metadata:
  type: feedback
---

Don't name a code path's role from one detail (an `fd`, a debug-looking flag). Look up where its
callees are defined first: `grep -rn func_X src` / the yaml segment.

**Why:** 2026-11-05 I called movie_cd.c's `m->unk5075D` branch a "host file" read in chat and in
the 0015B490 nonmatching note; its callees `func_00166970`/`func_001669B0` are in hddcopy.c, so it
is the HDD install. The note had to be corrected.

**How to apply:** for any "this path does X" claim, list the callees' TUs in the evidence.
Related: [[follow-field-copies]], [[state-evidence-at-real-strength]].
