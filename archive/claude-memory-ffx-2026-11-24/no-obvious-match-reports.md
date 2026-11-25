---
name: no-obvious-match-reports
description: Don't report that a pure constant/name substitution still matches; only flag semantic mismatches (e.g. enum doesn't fit the value it's compared with)
metadata:
  type: feedback
---

Don't tell the user that swapping a literal for a named constant/#define/enum (e.g. `2` → `SCECdComplete`) still matches. It obviously compiles the same (user, 2026-10-31: "of course it compiles to the same").

**Why:** noise. What they do want to hear about is meaning: when a constant or enum doesn't actually represent the value it's compared with or assigned to (wrong group, wrong function's return codes, flag vs value).

**How to apply:** skip match checks for pure renames/constant substitutions, or at least don't report them. Do check and report semantic fit. Related: [[cleanup-checklist]], [[flag-sets-as-enums]].

**Also (user, 2026-10-31):** don't run `make`/the checksum after adding comments, defines or constants that nothing uses yet; it wastes time and tokens. Build only when a change can affect codegen or linking.

**Also (user, 2026-11-08):** in notes, "matches" already means the full build is OK. A match record is the date, "matches" and the commit hash; nothing else (no word counts, "relocations checked", "full build (not) rerun"). Notes are in git, so the date is only for a quick look.
