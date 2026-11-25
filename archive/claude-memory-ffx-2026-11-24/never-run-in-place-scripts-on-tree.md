---
name: never-run-in-place-scripts-on-tree
description: "swapfn.py (and any helper) edits its input file in place; only ever pass it a scratch copy, never a path under the user's repo"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

swapfn.py writes its result back into the file it is given. On 2026-10-19 I passed it `src/battle/battle.c` directly and wiped the user's in-progress BattleActionType09 body (everything after their rewritten prefix). The user called it "a pretty big fuckup" and my reconstruction "garbage".

**Why:** the user's working tree is theirs ([[no-code-changes-without-permission]]); a live in-progress function has no git copy, so an overwrite can lose real work.

**How to apply:** before running any helper script, check whether it writes its input (read the script). Always `cp` the user's file into the scratchpad first and run tools only on the copy; no repo path ever goes in a write-capable argument. For check-ins on a live function, snapshot the file into scratch at every check-in so an exact copy exists. If something does go wrong, offer exact snapshots, not stitched guesses.

**FFX:** `tools/decompile.py <func>` injects m2c output into the C file in place; only `--print` is read-only. Run it on a scratch copy unless the user asks.
