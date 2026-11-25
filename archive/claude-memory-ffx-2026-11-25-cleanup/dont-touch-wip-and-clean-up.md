---
name: dont-touch-wip-and-clean-up
description: Never edit the user's in-progress function; side work goes in my own scratch space; remove worktrees/branches I create once they're redundant
metadata:
  type: feedback
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

- **Don't touch the function the user is working on** (user, 2026-10-23: "Just don't touch my function"). When a side change (e.g. a type refactor) is needed while they have uncommitted work, do it in a scratch worktree / scratch repo and test there; apply to their tree only when asked, and then edit only the other sites (check `git diff` per file first; staged vs unstaged keeps the two sets apart). "My worktree" from the user means their working tree.
- **Clean up after myself** (user, 2026-10-23: "Clean up after yourself man"): a `git worktree` + branch I created for side work must be removed (`git worktree remove --force <literal path>`, `git branch -D`) as soon as its content lives on the user's branch — don't hand the cleanup back as instructions.

**Why:** the user commits/stages between messages and owns their tree; leftover worktrees and branches clutter `git branch` and confuse later git state checks.

**How to apply:** before creating a worktree/branch, plan its removal; at the end of the side task verify the content is in the user's branch (`git show --stat`) and remove both. Related: [[silent-backups]], [[check-git-state-before-claiming]], [[no-code-changes-without-permission]].

**FFX, 2026-10-30:** I ran `git stash` / `git stash pop` on the user's tree (with their uncommitted read_debug.c work) to compare object files. It came back intact, but it's the same mistake: build comparisons go in a scratch copy (rsync of the repo), never stash or rebuild their tree for my own checks.

**WIP means a failing build (user, 2026-11-05):** if the tree has an unmatched WIP function, expect
`make build` to fail and don't investigate it. Only check my own change when needed (build or
compile it against HEAD's version of the WIP file), and say so in one line; no scratch re-split
debugging for the WIP itself.

**`make format` touches WIP too** (2026-11-24, header move during the user's MsBtlListInit WIP: it
reformatted their casts). When the user has unstaged WIP, format only the files I edited
(`clang-format -i <files>`), never `make format`.
