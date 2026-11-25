---
name: watch-battle-assembler
description: FFX battle folder builds with dtls13010 on 1-sign evidence; flag anything that contradicts it while working on battle code
metadata:
  node_type: memory
  type: project
  originSessionId: 2d64f870-2fa5-4917-92a3-e10aae28aa90
  modified: 2026-11-23T19:53:20.129Z
---

Since 2026-11-23 `src/game/battle` (and any folder in the Makefile's `DTLS_DIRS`) uses the dtls13010
assembler, chosen on one sign (`func_001AF188`) and none against (Toolchain.md, bullet under "Which
code used which assembler").

**Why:** the user accepted thin evidence to move forward and asked me to stay alert for contradictions.

**How to apply:** when matching or scanning battle code (incl. the `Ms*` camera once split), raise it
immediately if a function matches only with `-01` (variant.py `--asm dtls` vs default), or the target
has a fillable `P; jr ra/jal/b; nop` slot in a plain function. Also re-check after a folder is added to
`DTLS_DIRS` (needs a clean rebuild of those objects; they don't depend on the Makefile). See
[[ffx-matching-lessons]].
