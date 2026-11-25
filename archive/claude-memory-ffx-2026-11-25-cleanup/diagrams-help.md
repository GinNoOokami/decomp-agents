---
name: diagrams-help
description: The user finds layout diagrams very helpful; prefer horizontal boxed form (ArchivePrimer.md bit-layout style) when it fits, in code comments and explanations
metadata:
  type: feedback
---

Draw diagrams for layouts (buffers, structs, bit fields, address ranges) in explanations and put them
in the code's comments (e.g. above the struct in the header). Preferred: horizontal, boxed with
`+---+` borders and offsets/field names above the boundaries, like ArchivePrimer.md's file-table
word (`31 24 23 22 21 0` over `| padding / 8 |E |C | start LSN |`), as long as it stays within
~100 columns; vertical (one row per region) only when a horizontal one gets too long.

**Why:** 2026-11-05: the user said a buffer-layout diagram I drew in chat (SaTask: header, padding,
thread slots, save area, entries, with which field points where) helped a lot, and noticed I had
only put a one-line version into sa_task.h.

**How to apply:** when a layout is established, propose the horizontal diagram for the code comment
(still with approval). Pairs with [[explain-step-by-step]] and [[repo-self-contained]].
