---
name: pull-in-related-constants
description: "Defines/constants are the exception to \"only what's used\": when the first one of a set is needed, add the whole related group; prototypes/functions stay used-only"
metadata:
  node_type: memory
  type: feedback
  originSessionId: db671b69-bd50-4e94-b4b1-76c06a4f03c3
  modified: 2026-11-16T00:12:37.859Z
---

When a define/enum value is first needed, add every related value of that set at once (e.g. needing
`sceMcExecFinish` → add `sceMcExecRun`, `sceMcExecIdle`, `sceMcExecFinish` from the SDK libmc.h).
This is the only exception to "headers hold only what FFX is shown to use": prototypes, structs and
function names stay used-only.

**Why:** magic numbers are much harder to spot and infer when their definitions don't exist yet;
a missing function name doesn't hide anything the same way. Standing order the user had to repeat
(2026-11-15, libmc.h: I added only sceMcExecFinish and cited the header's "only what FFX is shown
to use" comment); they said I "continually ignore" it.

**How to apply:** the rule lives here, not in the repo: leave header comments like libmc.h's "Only
what FFX is shown to use" as they are; don't encode working rules in source comments. For SDK constants, copy the full group from ref/globe (or sce-sdk) headers with real
names/values; for game value sets, see [[flag-sets-as-enums]] and [[sony-sdk-samples]].
