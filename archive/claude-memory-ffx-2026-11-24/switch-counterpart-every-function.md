---
name: switch-counterpart-every-function
description: "FFX - run the Switch counterpart check for every function I review, match or name (swfunc.py, then functions.tsv + --sw when its row is missing/stale); shared.tsv is not the full list"
metadata:
  node_type: memory
  type: feedback
  originSessionId: cb63b8c7-128f-4616-90ab-ab8ade4e58e8
  modified: 2026-11-24T20:01:30.744Z
---

For every PS2 function I give matching tips, cleanup or names for, look up its Switch counterpart
first: `~/dev/ffx-analysis/scripts/swfunc.py FUNC`. If it has no row or an implausible one (swfill
rows are stale until swpair.py is rerun with new repo names), find the neighbours of the nearest
named PS2 functions in `~/dev/ffx-analysis/builds/switch/out/functions.tsv` (the full FFX Switch
list) and compare the candidate with `swfunc.py FUNC --sw ADDR` (stores/constants; callees in
`builds/switch/out/calls.tsv`). Never use `out/swx2/shared.tsv` as the Switch list: it only holds
exports shared with FFX-2.

**Why:** 2026-11-17, 2D2258.c: I reviewed nine pad helpers and said "no Switch name" from
shared.tsv, then proposed invented names. The user doubted they were all inlined; the full list had
`TkResetControlerWork`, whose decompile is PS2 0x2D38C0 plus those helpers inlined. The user wants
this as a standard step that every agent knows (file + tool).

**How to apply:** report the counterpart (or "none, inlined into X") in each review before naming
or cleanup; inlined helpers get named via their entry ([[switch-inlining-trap]]). Steps are in
DecompNotes `FFX/SwitchBuild.md` "Per function, while matching". Related: [[cleanup-checklist]],
[[naming-preferences]].

**"No swpair/swfill row" is not "no Switch name"** (2026-11-24, func_001AF238 = MsGetFileSize):
I stopped at swfunc's empty row and told the user it had no Switch name (twice: in the tips and the
cleanup). Next steps were open and cheap: a handoff lead already named the caller (func_001ADE50 =
MsBattleExe?), whose Switch body calls MsGetFileSize(0xe, 0xb, ...). Also, one caller's port being
reworked (getExcelBinSize by name) says nothing about the callee's other callers. Before saying
"none", read the Switch bodies of the known/lead-named callers for a call with the same arguments.

**Bulk pairing when swfill fails** (2026-11-19, libmscd, see SwitchBuild.md "libmscd names"):
function-pointer tables pair slot by slot, but 8-byte pointers shift offsets unevenly, so anchor
on known slots on both sides of the one you name; `movie_err`/assert line deltas between two
functions match across builds; constant-returning helpers can be checked by script (values equal).
A swfill row can be a pure size coincidence (FUN_0054c270 for func_001246B0 was script-VM code):
read its body before quoting it. Before a bulk rename, verify each pair against both bodies (user
asked for a recheck; it found one wrong pair and an overstated note).

**Run pairing (2026-11-19, tklib):** swfill rows in 124440-126108 were wrong ~every time, but reading
functions.tsv in address order beside the PS2 file paired whole runs body-for-body (getters/setters,
stubs between two anchors). Confirm a generic body (`return 0`, `call; flag = 1`) by position between
two body-matched anchors and by listing every Switch user of the global vs PS2 users (1:1). Inlined
helpers (MsGetSaveEventAddress, TkLoadVibrate parts) explain PS2 functions with no Switch row.

**Tool (2026-11-20):** `scripts/swrun.py FUNC` shows the PS2 run beside the Switch run with bodies; `--users PS2SYM SWSYM` for the global 1:1 check. Use it before saying a function has no Switch name.

**Alpha traces and data exports (2026-11-21, tklib retro):** `swfunc.py FUNC` now prints `// Alpha`
lines first: check them before writing any lead (three tklib functions print their own names there;
our leads ignored them). For globals, `functions.tsv` has no data: check `builds/switch/symbols.tsv`
OBJECT exports around the Switch addresses of neighbouring globals (missed `TkEventMapInfo`, made it
static, had to revert). Struct-field renames: grep `->field` across all of src/ first.
