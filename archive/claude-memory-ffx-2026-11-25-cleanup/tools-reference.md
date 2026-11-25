---
name: tools-reference
description: "FFX tool catalogue is DecompNotes/FFX/Tools.md - check it before writing any script, extend tools over one-offs, add every new tool"
metadata:
  type: feedback
---

The user wants no duplicate one-off scripts. `~/dev/DecompNotes/FFX/Tools.md` lists every FFX
tool by task (repo `tools/`, `~/dev/ffx-analysis/scripts/`, `DecompNotes/tools/`).

**Why:** (2026-11-01) I wrote `symuse.py` while `refs.py`/`xref.py` already covered part of it;
the user asked for a catalogue so we stop wasting time and tokens rebuilding tools.

**Rule restated (user, 2026-11-06):** always check whether an existing tool does the job; if not,
whether a similar script can be extended. One-off scripts only when 100% certain there is no reuse.
Missed on 2026-11-06: wrote a scratch caller-analysis script while `symuse.py` covered one symbol;
the fix was a batch `--exports` mode in symuse.py. The user tested it and confirmed (2026-11-06) this
is the tooling they want: small query options on shared tools that answer recurring questions
quickly for both of us, an efficient use of tokens.

Missed again 2026-11-10 (func_002D1EA0, session started from a pasted handoff after /clear): built a scratch compile/diff harness, a scoring script and a 720-order sweep while `variant.py` and `vsweep.py orders` already do this; my scorer's normalization also produced false diffs. A handoff that doesn't mention tools is no exemption: grep Tools.md before the first scratch compile. **Repeated 2026-11-22** (1AC068 VU0 matches): wrote run338.sh/cmpfn.py compare scripts (then debugged objdump `...` zero runs and mask leaks) instead of `variant.py FILE FUNC -f snippet.c` (`--repo out/scratchrepo` when headers change). Rule: every scratch compile of a candidate function goes through variant.py -f; no hand-made compare scripts.

**How to apply:** before writing a script or a repeated ad-hoc shell loop, grep Tools.md; prefer
extending an existing tool. Add each new tool (and mark superseded ones) in Tools.md in the same
step, then commit+push the notes. Repeated manual workflows (e.g. scratch variant compiles) are
tool candidates listed there.




**Region names and caller scans (2026-11-21):** name a caller region from
`ffx-analysis/data/systemmap.tsv` (or `symuse.py --runs`, which uses it), never by reading the
address (0x21xxxx below 0x21FC28 is battle UI, not menu; I put "menu" into the notes and had to
correct it). Scan callers/refs with `symuse.py` (reads the load image), not by parsing splat asm:
splat emits some code as `.word` + `alabel` (e.g. 0x1AC068-0x1ACEA8 in 171EC0.s), which a
`glabel`/`jal` text parse silently misses.

Matching-loop tool usage (variant/vsweep/rtl/perm, --repo, toolcheck, per-TU helpers) is in the
ffx-matching skill ([[ffx-matching-skill]]).
