---
name: warnings-must-be-deliberate
description: "FFX - NEVER mention the .space repeat-count warning (2D8658.c, known); every make: grep -v 'repeat count is zero'; no new warnings unless required for a match"
metadata:
  node_type: memory
  type: feedback
  originSessionId: c90454ab-162f-475d-a3c3-7efc470b931a
  modified: 2026-11-22T19:37:42.973Z
---

**FIRST RULE: every plain `make` I run pipes through `grep -v 'repeat count is zero'`; never write a word about that warning (3+ sessions of misses, last 2026-11-22 after the func_001AC408 apply: "stop telling me about that known warning").**

Any warning an edit introduces must be deliberate and required for a match (like the assembler's
`Warning: .space repeat count is zero, ignored` from splat output in tklib.c). Otherwise fix it
(prototype, include, cast, type) before proposing or applying.

**Why:** user, 2026-11-19, after `s32 TkGetAsia(void) { FFXGetAsiaMode(); }` was discussed (matches,
silent with build flags, warns under -Wall) and an implicit declaration slipped in via a missing
`devkit.h` include earlier the same day.

**How to apply:** after an edit, `touch` the changed C file, `make 2>&1 | grep -i warning`, compare
with the baseline (only the known `.space` one, see below). The build has no `-Wall`, so also
check implicit declarations by hand (every callee/global declared via a header or the file's
extern block); when a matching form needs a warning, say so in the proposal. Related
[[cite-sources-and-runtime-checks]], [[cleanup-checklist]].

**Reporting (user, 2026-11-19):** "Only tell me if there are new unexplained errors. You don't need to
inform me of the absence of any." Mention warnings only when a new, unexplained one appears.

**Known warnings (2026-11-22):** `Warning: .space repeat count is zero, ignored` from
`src/common/2D8658.c` (TkCheckMc's zero-length array) is required for a match. Never mention it.
`scratch.py --make` hides it via `KNOWN_WARNINGS`; in a plain `make`, filter it in the command itself
(`make 2>&1 | grep -i warning | grep -v 'repeat count is zero'`) so it never reaches me. Its temp-file
name (`/tmp/ccXXXX.s:1486`) hides the source file: don't investigate it or compare with HEAD.
Repeated miss 2026-11-22 (raised it after a rename build, user: "I really, really, need you to stop
bringing this up in every build summary"). When the user
accepts another required warning, add it to `KNOWN_WARNINGS` (per file) instead of re-reporting it.
