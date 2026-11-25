---
name: agree-requirements-first
description: "In design/cleanup discussions, settle requirements with the user before raising implementation, branching or \"shall I write it\" questions"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

When the user opens a design or cleanup topic (e.g. the attack-range defines, 2026-10-14), discuss the requirements until they say they're agreed. Don't push toward implementation: no "before I write anything", no branch/base questions, no offers to apply it.

**Why:** the user said "I don't want you to write anything yet, I want to talk about this and agree on the requirements before even talking about how to move forward" after I kept asking where to base the branch.

**How to apply:** answer the design question asked, raise open design questions if useful, and leave logistics (branching, PR order, who edits) until the user moves on to them. Related: [[no-code-changes-without-permission]].
