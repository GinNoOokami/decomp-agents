---
name: line-comments
description: "FFX: our own lib/game code uses // comments, never /* */; SDK headers (include/sdk) keep Sony's block style"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 81622c2b-18f5-4deb-9561-df22cc12a0b4
  modified: 2026-11-06T01:53:47.353Z
---

In our own lib and game code (src/, include/ outside include/sdk/), write comments with `//`, never
`/* */`, including multi-line function comments and section labels. Don't copy block style from
the file you're in. SDK headers (include/sdk/) keep block style, including comments I add there.
Struct offset tags `/* 0x10 */` are a separate convention and stay.

**Why:** the user prefers line comments (2026-11-05). I had copied `/* */` from libcdvd.h and
movie.h into the DiskReady defines and a function comment, and they asked why, then said: leave the SDK ones alone, prefer // for all our own lib and game code.

**How to apply:** all comments I propose or write in ~/dev/ffx. Leave existing block comments
alone unless asked ([[user-comments-are-theirs]]). Related: [[cleanup-checklist]].

Each comment must read on its own: no "it" / "when it did" that leans on the comment above
(user, 2026-11-05, `// sceCdGetError() when it did`: "The heck does this mean?").
