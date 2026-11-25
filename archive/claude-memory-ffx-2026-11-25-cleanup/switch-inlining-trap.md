---
name: switch-inlining-trap
description: FFX - Switch functions can contain inlined PS2 statics/helpers; check size ratio and PS2 neighbours before giving a Switch name to a PS2 function
metadata:
  node_type: memory
  type: feedback
  originSessionId: db671b69-bd50-4e94-b4b1-76c06a4f03c3
  modified: 2026-11-16T02:32:59.733Z
---

Before naming a PS2 function from a Switch candidate (swfunc/swfill), compare sizes. If the Switch
function is much larger, check the PS2 callees and neighbours for code it absorbed; the name goes
to the PS2 function with the whole Switch function's role (the exported entry), never to an inner
helper. Inlined statics have no Switch export at all.

**Why:** 2026-11-15 I gave `TkMcSync` (Switch 544 B) to PS2 0x2D86A8 (176 B), dismissing the size
gap as a "port-only wait loop"; the loop was PS2 0x2D8758, which had inlined 0x2D86A8. The user had
to have the name moved and asked me to note it so I don't fall for it again.

Repeat 2026-11-18: I named PS2 0x2D38C0 `(port, slot)` `TkResetControlerWork` although the Switch
function takes no arguments and resets both ports; the entry was the PS2 wrapper 0x2D3990 (no args,
loops ports/slots calling 0x2D38C0). Caught one function later; renamed back with an amend.
Signature mismatch (arg count, all-ports vs per-port) is the same signal as a size gap.
Repeat 2026-11-19, via a string pair: swpair paired Switch `printf2` (688 B, variadic) with our
`va_list` worker DebugVprintf (528 B) because the strings live there; the entry was our variadic
DebugPrintf (56 B), which both builds install with `sa_printf_set`. Found only when the user asked
for a recheck after the bulk rename. String pairs are not immune: check the signature too.

**How to apply:** before proposing any Switch name, compare signatures and check the PS2 callers
against the Switch callers (`builds/switch/out/calls.tsv`, through export stubs); treat a size flag (`size,weak`) as a prompt to check inlining first, not as a port
difference. Details and the observed cases: DecompNotes `FFX/SwitchBuild.md` "Inlining trap".
Related: [[add-symbol-names-for-user]].
