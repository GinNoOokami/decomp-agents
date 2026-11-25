---
name: sdk-heavy-cheat-ok
description: For SDK/hardware-heavy functions the user may ask me to apply the full match; tools may manage their own marked output dirs
metadata:
  type: feedback
---

(1) 2026-11-07, func_001581E0 (GS packet builder): the user said "I'm happy to cheat on this one, go
ahead and apply your match" because it relied on SDK/graphics types they didn't know, and they
didn't want to chase unnatural C shapes. The default stays [[no-unrequested-decomp-answers]]; this
is per function, on their request. When applying: repo types, SDK macros/constants (whole groups,
[[ffx-repo-conventions]]), natural shapes (unions over casts), list what's still raw.

(2) Same day: "Can you have the tool itself delete the repo directory? I don't want to micromanage
a copy of the repo every time it needs a refresh." Tools may delete a directory they created
themselves, guarded by a marker file they wrote (toolcheck.py's `repo/HEAD`); never anything else.

**Why:** the user's time goes to reconstruction, not to SDK trivia or housekeeping.
**How to apply:** offer the full match only when asked; for refreshable tool outputs, build the
self-replace in instead of asking the user to move or delete things.
