---
name: header-prototypes-external-only
description: "FFX header placement is a default, not a rule: codegen evidence of the original's declarations (per-TU prototypes, per-TU global types) overrides it; otherwise headers declare only functions called from other TUs"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 81622c2b-18f5-4deb-9561-df22cc12a0b4
  modified: 2026-11-06T04:17:50.182Z
---

Put a prototype in an area header (`include/lib/movie.h` etc.) only if some other TU calls the
function, in C now or in asm that will become C. A function called only inside its own TU (could be
`static`) gets no header entry: declare it near the top of its .c file if a C use comes before the
definition (e.g. its definition is in a `NON_MATCHING` branch), otherwise nothing.

**Why:** user, 2026-11-05 (movie_cd.c): "As long as they're needed eventually, that's fine. ... I just
don't want to add a header prototype that is only used within a TU (could be static)".

**How to apply:** before adding a prototype, check callers: `grep -rlE "(jal|j) +NAME\b" asm/jp_intl`
plus C uses outside the file (`symuse.py NAME`, or `symuse.py --exports --tu lib/<file>` for a whole TU).

**Placement map (user, 2026-11-06, moviefunc_ac.c cleanup):** lib function called across lib files →
`include/lib/sasa.h` (one header for the whole lib, sections per source file / unsplit blob); game
function called from lib → its owner's header if the owning file/system has one, otherwise an
`extern.h` area group. **`extern.h` is only a holding area** for externs whose home isn't known or
doesn't exist yet (user, 2026-11-06): never treat it as a destination; when a file is split or a
system header is created, move its group out, and flag `extern.h` entries whose home now exists; file-local data externs and TU-only prototypes
(e.g. devkit addresses above 32 MB) → one block at the top of the .c, data sorted by section then
address, never scattered before each function. The user asked for "externs to the top of the file,
or headers where appropriate" and approved this split. Related: [[ffx-repo-conventions]], [[follow-field-copies]].

**Evidence overrides this (user, 2026-11-17):** "That guideline was proposed with the assumption the
developers practiced clean header organization. We have evidence to the contrary now, so obviously
we shouldn't apply any hard and fast rules when it comes to source shape or project structure."
Evidence found: read_debug.c called `TkSignalSemaBGThread` with no prototype (implicit int, v1 after
the call) while libmscd.c and hddcopy.c saw `void` (v0). So lib files declared game/TK symbols per
file. (`lb` vs `lbu` loads alone are NOT signedness evidence: `volatile s8` still compiled to `lbu`.) **How to apply:**
when codegen shows how a TU declared something (call-result register, signedness, prototype
presence), reproduce that per TU even if it means local extern blocks or missing prototypes; the
placement map above is only the fallback when there's no such evidence.
