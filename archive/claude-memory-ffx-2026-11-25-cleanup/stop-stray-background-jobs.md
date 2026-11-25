---
name: stop-stray-background-jobs
description: "A command auto-moved to background (timeout) keeps running; stop it with TaskStop once it's unneeded or wrong"
metadata:
  node_type: memory
  type: feedback
  originSessionId: c85318ad-ec8e-41c1-b995-b176838c4e0d
  modified: 2026-11-10T06:17:48.803Z
---

When a Bash command times out it becomes a background task and keeps running. As soon as it's unneeded (answer found elsewhere) or wrong (e.g. a `find /` crawling `/mnt`), stop it with TaskStop right away and say so.

**Why:** 2026-11-09, func_0011F2B8: a `find /` for haifa-sched.c went to the background, crawled the Windows drives (breaking the no-/mnt-search rule), and ran to completion. I found the path in memory seconds later but never stopped it. The user asked why I didn't.

**How to apply:** track every background task ID I start; when I abandon its purpose, TaskStop it in the same turn. Related: [[ffx-matching-skill]], [[never-block-on-background-jobs]].
