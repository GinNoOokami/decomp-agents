---
name: check-own-side-effects
description: "Before attributing a working-tree change to the user, check whether a command Claude ran (make, splat) wrote it"
metadata:
  node_type: memory
  type: feedback
  originSessionId: d8b8ba71-d5b2-44ee-a8a4-83ea356c0bf1
  modified: 2026-10-31T01:45:28.952Z
---

A modified file in `git status` may come from Claude's own commands: in `~/dev/ffx`, `make build` can re-split, and splat rewrites `include/include_asm.h` etc. On 2026-10-29 Claude told the user an `include_asm.h` change was their edit; it was splat, run by Claude's `make build`.

**Why:** wrong attribution misleads the user about their own tree and wastes their review time.

**Don't create them either (2026-10-30):** gcc `-da` writes dump files into the current directory
(20 `r6.c.*` files landed in `~/dev/ffx`), and a quick comparison wrote `.r.txt`/`.ro`/`.da` next
to an `expected/` object. Run compilers with `cwd` = scratch and write every temp output under the
scratchpad; check `git status --untracked-files=all` after such commands.

**"Changed on disk" notices (2026-10-30):** a file-changed notice that arrives right after my own
`sed`/python edit of that file is my edit. I told the user they had edited `ProjectLayout.md`;
it was my `sed`. Before saying the user changed a file, check whether my last commands wrote it.

**How to apply:** compare the file's mtime with the commands just run, and know which tools write which files before saying who changed what. Say "my `make build` re-split and splat rewrote X" when that's the case. See [[state-evidence-at-real-strength]].

**Tool runs (2026-11-04):** when I run the ffx-analysis tools myself (variant, vsweep, perm, dme, rtl,
toolcheck), pass `--out <my scratchpad dir>` and delete it afterwards; the defaults (perm/, decompme/,
out/...) are the user's working areas and I'm not allowed to delete there.
Put deletions in their own small command, never at the end of a batch (a bundled `rm -r` got the
whole batch refused, 2026-11-04); if a scratchpad delete is refused, leave it (session-only).
