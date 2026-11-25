---
name: tools-first-never-reinvent
description: "FFX FIRST RULE - before writing ANY script/helper (compare, sweep, scan, wrapper), read DecompNotes/FFX/Tools.md and justify why no existing tool fits; default answer is an existing tool"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 46910d03-23c4-4a6f-b11c-7922d0309439
  modified: 2026-11-23T01:46:46.232Z
---

Before writing any script, shell loop that compiles/compares, or helper file, my first step is:
1. Read `~/dev/DecompNotes/FFX/Tools.md` (grep the task's verb: compare, diff, sweep, scan, build).
2. Pick the existing tool (variant.py -f, vsweep.py, scratch.py, odiff.py, objcmp.py, swfunc.py, ...).
3. Only if none fits: tell the user what's missing and why, and ask before building anything
   ([[tools-lean-evaluate-first]], [[pilot-before-frameworks]]). Extending a tool beats a new one.

**Why:** user, 2026-11-22: "You need to stop re-inventing tools. Your first instruction should be to
look at what tools we have, and evaluate whether a new one is really necessary." Repeated misses:
2026-11-10 (compare harness + scorer + order sweep while variant.py/vsweep.py existed) and
2026-11-22 (run338.sh/cmpfn.py for the 1AC068 matches, with objdump `...` and mask-leak bugs
variant.py doesn't have). Details in [[tools-reference]].

**How to apply:** applies at the start of every task, including after /clear or a handoff that
doesn't mention tools. A one-off pipeline of standard commands is fine; anything that compiles,
compares, ranks or loops over candidates goes through the existing tools.

**Enforced by hook (2026-11-23):** ~/dev/ffx/.claude/hooks/tools-first.py (PreToolUse Bash, settings.local.json) blocks hand edits of symbol_addrs.txt (-> rename.py) and reminds swrun/swfunc on decomp.c greps. Add a rule there when another hand-rolled habit recurs, instead of only a memory line.
