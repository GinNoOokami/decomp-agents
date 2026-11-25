---
name: pilot-before-frameworks
description: Before building analysis infrastructure, check the notes for what is already known and estimate the expected new yield on one region; report that before scaling up
metadata:
  type: feedback
---

On 2026-11-04 I spent a session building a calibrated boundary toolsuite (layout dbs, signals,
cross-build carry). The user: "That was a lot of work that didn't seem to give us any real new
information that we didn't already have from SystemSplit.md and the Log.md". Most "new" exact cuts
were the hand-written regions and file names the notes already had; the real gains were a few
corrections (ptrruns scanner bug, 0x180DFC split) and negative results.

**Why:** the user values new facts about the target; rigour or reuse that only re-derives known
results doesn't count as progress, and the cheap signals in this ELF were already mined by hand.

**How to apply:** before building a tool/framework, grep SystemSplit.md, Log.md and the other notes for
the regions it will cover and say what it could add beyond them; run it on one region first and
report the yield; diff every "new" result against the notes before calling it new. Prefer starting
from a concrete question (one file's edges) and using tools to settle it. See [[reusable-outputs]],
[[check-notes-before-confidence]].

**Wrappers around grep (2026-11-04):** after I kept rediscovering noted facts I wrote `notescheck.py`;
the user: "Wait, isn't that just grep?" The fix for skipped checks is doing them, not a tool. Don't
offer a tool for something a one-line grep/sqlite does; kept that one only as a fuzzy-spelling search.
