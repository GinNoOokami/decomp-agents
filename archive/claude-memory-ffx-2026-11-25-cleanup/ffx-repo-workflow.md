---
name: ffx-repo-workflow
description: "How to build ~/dev/ffx with the user - one piece per step, explain each piece's job, they write some, Claude drafts boilerplate, test in a scratch copy first"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 48b2336b-2a8e-40ab-ad53-efe697ab3ca1
  modified: 2026-10-29T07:17:58.007Z
---

The user is building `~/dev/ffx` from scratch to learn how each piece works (first time setting up
splat; less familiar with Make, fluent in Python and C++). Agreed way of working (2026-10-28):

- One piece per step, in order; say what the piece is responsible for and why before the details.
- They write some parts themselves; Claude drafts boilerplate and refines what they write
  (maintainability, extensibility), always listing every change vs their version.
- Before handing over a config or Makefile change, prove it in a scratch copy of the repo
  (e.g. milestone 0 was reached in scratch first, then walked through step by step). Report the
  fixes that testing needed; don't present untested config as working.
- They review staged files and ask for a check before committing; they commit and push themselves.
- Once they've reviewed a change in chat they often say "go ahead"/"make those changes": then apply exactly the tested version (backup first), show `git diff` is only that, and run `make` in their tree to confirm.
- A bare shell command typed as their message (e.g. `git rm objdiff.json`) means run it.

**Why:** they want understanding, not a scaffold; untested proposals cost them debugging time.

**How to apply:** pair with [[ffx-repo-conventions]], [[small-files-inline]],
[[repo-self-contained]], [[readme-upkeep]].

**Scratch checks (2026-10-30):** `rsync -a --delete --exclude build ~/dev/ffx/ $S/<name>/`, edit there, `make` (sha1 ✓/✗). `odiff.py` is hardwired to `~/dev/ffx`, so in scratch compare a function with `mips-linux-gnu-objdump -d --start-address=… --stop-address=…` on the scratch ELF against the words in the target `.s`. Never stash or rebuild the user's tree for my checks ([[dont-touch-wip-and-clean-up]]).

**Scratch rebuilds after Makefile edits:** objects don't depend on the Makefile, so a scratch
build after a Makefile change reuses stale objects (cost a false failure 2026-11-04): delete the
scratch `build/` (or `make clean`) first, and tell the user to do the same after applying.

**No scratch build for cosmetic changes (user, 2026-11-04):** moving declarations, comments,
renames: still only with the user's explicit approval for that change (show it in chat first);
once approved, apply in the tree and run one `make build` there instead of a scratch build. Scratch copies are
for config/Makefile changes and matching experiments. Conserve tokens.
