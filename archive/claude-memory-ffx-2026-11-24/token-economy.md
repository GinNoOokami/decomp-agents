---
name: token-economy
description: "Keep context small - filter tool output, write big results to files, grep Log.md instead of reading it whole"
metadata:
  node_type: memory
  type: feedback
  originSessionId: f9744cf8-80ad-4ec4-ae82-43f7bb0923b8
  modified: 2026-10-29T07:09:20.676Z
---

Keep the conversation context lean; every turn re-sends it.

- Filter tool output before it lands in context: pipe objdump/asm/m2c/asm-differ/diffs through grep, sed -n, head, or a small script that prints only the function or changed lines.
- Big results (scans, symbol tables, cross-version alignments) go to files in the work folder; report counts plus a few example lines, read back slices.
- Never read DecompNotes/FFX/Log.md whole; grep it or read the sections the handoff names.

**Why:** User asked to cut token usage (2026-10-29). Fixed per-session overhead is tiny; context growth from dumps is the real cost. The user already clears at milestones and works one function per session.

**How to apply:** Default to filtered/sliced reads in every step. Prefer direct grep/sed over subagents. See [[keep-working-log]], [[handoffs-not-in-notes]].

**Budget pressure (2026-10-30):** user had used ~60% of the weekly token budget by day 3. Ask before starting expensive optional work (interpreters, exhaustive passes); offer the cheap option first.
