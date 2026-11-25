---
name: ff7-decomp-reference
description: The user's previous decomp project ~/dev/ff7-decomp is their reference for tooling/workflow; read its implementation before describing how it behaves
metadata:
  type: reference
---

`~/dev/ff7-decomp` (Xeeynamo's FF7 PSX decomp, which the user contributed to) is the user's model for project tooling: `make build` (Go builder in `tools/builder/`, ninja, sha1 check, then `expected/` refresh), `make format` (pinned clang-format + black), `./mako.sh dec <func>` (`tools/decompile.py`, m2c injected over `INCLUDE_ASM`), `make lint` (clang-tidy + Go lints). FFX equivalents were built from it on 2026-10-29.

**Why:** the user asks "how did FF7 do it?" and remembers its behaviour from use. Twice this session a claim about it was wrong until its source was read: the build-failure workflow (it also stops at the sha1 check, keeping `expected/`) and `clean` (it removes `asm/` too, so it re-splits).

**How to apply:** when comparing, read the actual code (`tools/builder/builder/*.go`, `tools/ninja/gen.py`, `tools/decompile.py`) and quote it. Read-only; it's not ours to edit. See [[ffx-repo-workflow]].
