---
name: add-symbol-names-for-user
description: "FFX - name only MATCHED functions (C in the tree, never INCLUDE_ASM); agreed names: I edit symbol_addrs.txt + C rename myself"
metadata:
  node_type: memory
  type: feedback
  originSessionId: db671b69-bd50-4e94-b4b1-76c06a4f03c3
  modified: 2026-11-16T00:17:15.760Z
---

Once a name is agreed (e.g. a Switch candidate confirmed by logic comparison), I add it to
`config/jp_intl/symbol_addrs.txt` (in place, address order, `// type:func original name, from ...` naming the evidence source, matching the existing wording; Switch: "original name, from the Switch port's exported symbol" (no 'same logic' tail, user 2026-11-15)), rename the C
definition/references, run `make`, and check the asm callers picked it up.

**Why:** user, 2026-11-15 (TkMcSync): "I always want you to add the symbol names for me."

**How to apply:** standing permission for symbol_addrs edits and the matching rename only; the
name choice itself is still the user's ([[user-verifies-findings-before-pr]]). Naming cadence: name
as we go for functions with a Switch candidate, batch the rest at the end of the TU. Only name a
function once it is matched (C in the tree): 2026-11-15 I moved `TkMcSync` onto the unmatched
0x2D8758 when the user only wanted the wrong name removed from 0x2D86A8. Removing a wrong name ≠
moving it. Repeated 2026-11-17: asked to "apply the renames", I named three INCLUDE_ASM
functions (TkOpenControler, TkCloseControler, TkResetControlerWork) and had to revert them; check
`grep INCLUDE_ASM` for each address before writing a name, and keep unmatched leads in chat or the
handoff instead. Before naming from a Switch export, check whether neighbouring PS2 functions were inlined
into it (Switch TkMcSync = PS2 0x2D8758 + inlined 0x2D86A8; PC kept both).

**Passthrough wrappers: apply directly (user, 2026-11-15):** for predictable passthrough cleanups
(e.g. the TkMc* wrappers: Switch name + SDK parameter names/types), apply the cleanup and the name
without proposing first; report what changed. Anything with a judgement call (types beyond the SDK
prototype, constants, structure) still gets proposed.

**Tool (2026-11-20):** use `scripts/rename.py OLD NEW` (dry run `-n`; `--data --attrs ...` for globals) instead of hand sed; it refuses INCLUDE_ASM functions and runs make. Switch names: always `--switch` (adds the marker, checks functions.tsv); 2026-11-21 I ran 13 Switch renames without it and the lines kept a bare `// type:func`.
