---
name: persistent-scratch-repo
description: "FFX - full-build checks use one persistent scratch copy synced with rsync (~/dev/ffx-analysis/out/scratchrepo), not a fresh cp -a of the 171 MB repo each time"
metadata:
  node_type: memory
  type: feedback
  originSessionId: c90454ab-162f-475d-a3c3-7efc470b931a
  modified: 2026-11-20T08:46:12.126Z
---

For "does this still build/match" checks on proposed edits, sync one persistent copy and build there:

    scratch.py -e edits.py [NAME] --src src/...c --make   # ~/dev/ffx-analysis/scripts
    scratch.py --fn def.c --func NAME --src src/...c --make
    # edits file = variant.py format, so the same file feeds variant.py -e and vsweep --repo

(2026-11-21: user approved scratch.py after the retro found ~8 hand rsync+edit-script rounds.)

Incremental (keeps build/), nothing to clean up afterwards, never touches the user's tree. Tools that
take FFX_REPO / --repo can point at it too (also covers [[analysis-tools-from-head-copy]] when
synced from a HEAD export instead).

**Why:** retro 2026-11-20 ([[session-mini-retro]]): about 7 `cp -a` copies of the whole repo in one
session for single-function checks; user: "Save the habit change idea for 4 to memory".

**Before writing scratch edits, check `git diff` for the user's WIP on that function** (2026-11-24):
their draft had replaced the `INCLUDE_ASM` line, so my anchor failed; anchor on their current text.
Keep the `func_` name in test variants until the rename (the asm callers still link against it).
