---
name: try-compiler-flags-when-stuck
description: "When a match is stuck on instruction order/scheduling that no natural source fixes (siblings match), test compiler flags in scratch, -fno-strict-aliasing first, with a whole-project diff"
metadata:
  node_type: memory
  type: feedback
  originSessionId: a69d1169-1fcc-4ea3-97c9-ca7227bb20b9
  modified: 2026-11-03T02:26:58.143Z
---

When a function is stuck on a codegen detail that survives every natural source shape (especially the order of independent loads/stores, or needs an odd type trick like a one-member union or char access), test compiler options in scratch before more rewrites. Start with `-fno-strict-aliasing`, then the other `-f` flags gcc 2.95 sets at `-O2` (toplev.c `optimize >= 2` block). For any candidate, compile every C file with and without it and diff the disassembly; adopt only if nothing else changes.

**Why:** 2026-11-02, func_00164320 stuck at 190/194 (callback reload vs `read_q_req_no` load order); dozens of source variants did nothing. Tracing `haifa-sched.c`/`alias.c` showed it was alias-set based, and `-fno-strict-aliasing` matched it with zero side effects project-wide; it also explained the "tentative union" in `ReadQEntry.buff`. The user asked to keep this in mind "for future situations where matching is seemingly impossible". The project now builds with `-O2 -G0 -fno-strict-aliasing` (evidence in DecompNotes FFX/Toolchain.md "Options: strict aliasing", case 2 in FFX/Troubleshooting.md).

**How to apply:** after the source/RTL trace ([[gcc-sources-before-rewrites]]) says the difference comes from an option-controlled analysis (alias sets, cse-follow-jumps, scheduling), test the flag. A flag is a hypothesis about the whole build, not a per-function hack: whole-project diff, then report to the user (they edit the Makefile). Flags can be per library (libmscd may differ from game code), so retest new files both ways. Not a license to skip real structure ([[no-fake-matches]]). variant.py hard-codes its CFLAGS; keep it in sync with the Makefile.
