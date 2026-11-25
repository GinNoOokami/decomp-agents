---
name: permuter-low-value
description: Permuter is a last-ditch tool, only once structure and types are believed right and a small codegen gap remains; never a main step or reported as progress
metadata:
  type: feedback
---

The permuter can be used, but only as a last-ditch step: when we're confident no large foundational
or structural issues remain (control flow, types, struct layout, statement shape all believed
right) and what's left is a small codegen detail. It is not a way to discover structure.

**Why:** user, 2026-11-05 (func_0015B490): "I don't count on the permuter in general, it's a hail
mary and I've rarely seen it turn up anything useful", then: "The permuter _can_ be used, but it's
more of a last ditch effort when we're confident there aren't any large foundational or structural
issues remaining." That day I ran it on a version whose loop structure was still wrong; its best
output was a fake `do { } while (0)` (diagnostic only).

**How to apply:** before suggesting it, state why the structure is believed settled; prefer RTL
dumps, gcc source and probes for anything structural. Treat its outputs as hints to explain, not
answers, and never present a running permuter as progress. Related: [[no-fake-matches]],
[[score-is-not-structure]], [[usage-budget]].
