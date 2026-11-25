---
name: answer-before-switching
description: "when a new request arrives mid-turn, finish answering the pending question first"
metadata:
  type: feedback
---

On 2026-10-30 the user asked how our names compared with the reference project, then sent "Ready for changes on read_debug_c" mid-turn; I switched to the branch work and the comparison answer never reached them ("Did you actually answer my last question?").

**Why:** mid-turn messages arrive while earlier questions are still open; switching silently drops the answer.

**How to apply:** when a new request lands mid-turn, either give the pending answer first or explicitly carry both in the final message.
