---
name: troubleshooting-entries
description: "FFX: offer a Troubleshooting.md case only for repeated m2c/build/matching problems the user hits, with a verified fix; one-off traps stay in my memory"
metadata:
  type: feedback
---

When a matching session turns up an m2c misreading or build problem with a fix that will recur
(e.g. a global called as a function = struct of function pointers; undeclared `sp`/`sp0` = local
passed by address), offer to add it as a case in `~/dev/DecompNotes/FFX/Troubleshooting.md`.

**Why:** 2026-11-06 the user suggested both of those entries themselves, right after I explained
each one; they want these patterns written down for next time.

**How to apply:** verify the fix with a compiled probe first ([[verify-hints-before-giving]]), then
write the case in the file's format (Symptom, Context, Cause, Fix, Later, Recognise it by), commit
and push. The file is grouped by symptom with an index at the top (user, 2026-11-06): a new cause
for a known symptom becomes a sub-case (4a/4b) with a "tell them apart" line; a new symptom gets
the next number in its group; update the index either way. One-off quirks of a single function don't qualify. Related: [[decomp-notes-docs]].

**The doc is the user's learning aid, not my compiler notebook (2026-11-08):** it holds the user's
recurring mistakes and m2c/build artifacts. gcc-internals findings (alias rules, cross-jumping,
reg-alloc causes) go in my matching memory ([[ffx-matching-lessons]]); don't offer those as cases.
The user's verdict on the struct-access alias rule: "That's the type of thing I would expect to go
in your matching memories".

**Only problems the user keeps hitting (2026-11-16):** the user wants the list kept short and focused
on repeated problems they run into while decompiling and matching. A one-time build or tooling trap
(e.g. the jtbl `align:0x10` fix) goes in my memory so I recognise it, not in the doc: "a bit niche
for an entry". Offer a case only once a problem has come up more than once, or clearly will.

