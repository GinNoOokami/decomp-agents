---
name: read-code-before-describing
description: Read the actual branch/body before stating what code does; three wrong claims in one session came from summarising from memory or a glance
metadata:
  type: feedback
---

Before telling the user what a function or branch does (which way a flag gates, what a size gap
means, why a symbol sits elsewhere), read the code or data that shows it, in that turn.

**Why:** 2026-11-15 I stated three things the code contradicted and had to correct each later: the
Switch `TkMcSync` wait loop was "port-only" (it was PS2 0x2D8758), the Asia flag "gates HDD on"
(it turns HDD off), and `ffx_pcmode`/`ffx_replaceiopimg` were "port additions" (they were PS2
bytes placed apart by initial value). Each went into a note before it was caught.

**How to apply:** quote the deciding line (`if (FFXGetAsiaMode()) enabled = 0;`) when describing
logic; for "why is X different" claims, check the obvious mechanical causes (inlining, section by
initial value, const) before a semantic story. Related: [[state-evidence-at-real-strength]],
[[switch-inlining-trap]].
