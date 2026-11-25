---
name: gcc-sources-before-rewrites
description: "When a codegen difference resists two or three rewrites, trace it in the gcc 2.95 sources and RTL dumps instead of more trial variants"
metadata:
  node_type: memory
  type: feedback
  originSessionId: fd20e0b1-093f-4206-9fbf-e08ced126534
  modified: 2026-10-31T07:55:38.807Z
---

When a match is stuck on a structural/codegen detail, go to the compiler: RTL dumps (`-dS`, `-dj`, `-dR`, `-da`) of my variant plus the pass source in `~/dev/ffx-analysis/gcc/gcc-2.95.2-ps2linux-3a/gcc/` (`cse.c`, `jump.c`, `reorg.c`, `haifa-sched.c` (ee-gcc is built with `-DHAIFA`; `sched.c` is unused), `gcse.c`, `loop.c`, `local-alloc.c`), to learn which source property produces the target's pattern. Don't keep generating rewrite variants and scoring them.

**Why:** 2026-10-31, func_00163950 / func_00255158: I ran ~15 rewrite/flag variants over a whole session (83–90%, none natural) and never opened the gcc sources, though I had proposed a `-dS` check early. The user asked: "Is there a reason you didn't use the sources to help answer questions and trace the problem?" Reading one pass is cheaper than many variant builds and answers *why*, not just *how much*.

This was already the rule from FF7 (ff7-decomp memory `gcc272-sources-and-operand-order`: "list which rules could apply, work out the tree shape each one needs, then write C for only those shapes; blind variant sweeps and the permuter never found this one"), and the user notes matching went much better there once I worked that way. FFX sessions don't load the FF7 memories, so the habit didn't carry over until this note.

**How to apply:** after two or three variants fail on the same detail, stop and trace it in the dumps/source; report what the pass needs, then test only the source shapes that satisfy it. Being stuck means switching to the source, never recommending INCLUDE_ASM ([[finish-matches-dont-suggest-stopping]]). Related: [[ffx-matching-lessons]], [[no-fake-matches]], [[verify-hints-before-giving]].

**Stubborn matches: any means, no source of ideas ignored (user, 2026-11-22):** no rigid order and
no gates; all else equal, cheaper sources first (a lookup before a deep trace). Use whatever helps: gcc source and RTL, other builds (PC, Switch, alpha), refs, matching
lessons, permuter hints, the stuck statement's natural positions. Before parking, though, every
one of them must have been looked at; a skipped one has to be named in chat. Also turn each trace and
diagnostic into concrete source shapes and test them.
**Why:** func_002D2890 (PC never opened) and Sg_ScreenshotWrite (trace and permuter hint never turned
into the obvious source test) were both parked with one source unused; another agent matched both.
See [[tried-tables-held-constant]], [[pc-counterpart-for-boundaries]].
