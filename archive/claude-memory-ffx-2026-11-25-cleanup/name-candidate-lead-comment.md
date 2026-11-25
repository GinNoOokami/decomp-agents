---
name: name-candidate-lead-comment
description: "FFX - matched function with no Switch symbol + a name candidate -> propose a `// Name?` lead comment above it, keep the func_ name"
metadata:
  node_type: memory
  type: feedback
  originSessionId: c90454ab-162f-475d-a3c3-7efc470b931a
  modified: 2026-11-20T05:31:41.050Z
---

When a matched function definitely has no name in the Switch symbol list (checked: functions.tsv /
decomp has no counterpart, not just a bad swfill row) and we have a descriptive name candidate,
propose a one-line lead comment directly above the definition, the user's form:

```c
// TkDrawDebugPacket?
void func_00124F68(void) {
```

The symbol stays `func_XXXXXXXX` (no symbol_addrs/C rename). Include it in the cleanup proposal.

**Why:** user, 2026-11-19 (func_00124F68, debug draw path stripped from both builds): "anytime we
have a name candidate that definitely does not have a name in the switch symbol list, we should
provide a leading comment like I've just done".
**Globals too (user, 2026-11-19):** "We should add leading comments to globals we're certain of the
names as well": a `D_` global whose meaning is settled gets a side-by-side `// gTkName?` comment on its extern line (user: "Do them side by side"; clang-format aligns them)
(repo style for our global names: `gTk...`, e.g. `gTkMainSema`), D_ name kept.

**No ready candidate is not a reason to skip (user, 2026-11-23, func_001AF7E8):** build one from the
system's own vocabulary (Switch names of the callee/caller family, e.g. `MsBtlRead*` + `MsMenuReadText`
→ `MsBtlReadMenuText?`); the `?` carries the uncertainty. Don't offer a role-description comment instead.

**How to apply:** part of [[cleanup-checklist]] step 4a's naming line; shape per [[lead-comment-shape]];
names with Switch evidence still go through [[add-symbol-names-for-user]].
