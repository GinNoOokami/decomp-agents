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

**Scratch copies with header edits:** pass `--repo SCRATCH` to variant.py/vsweep.py; without it they compile against ~/dev/ffx/include and moved prototypes become implicit declarations (a false 4-word mismatch on 2026-11-06). facts.py has no such option yet.

**Per-TU / per-session tools (2026-11-06):** `nextfunc.py lib/<tu>` picks the next target (start of a matching session); `tidy_externs.py <c> <func>` places m2c's declaration block (offer it after each new match); `decompile.py FUNC --print --gotos-only` shows which value each guard path returns when m2c's nesting hides it. Test changes to these on scratch copies (`--repo`) before the user runs them: two bugs reached the user's tree this session (--apply crash, a dropped forward prototype).

**After changing any per-function tool** (ffxfunc, variant, facts, perm, dme, rtl, vsweep): run
`~/dev/ffx-analysis/scripts/toolcheck.py run` (106 known matches, all must pass; "known" = unpaired
lui) before using the tool's results. Its first runs (2026-11-04) found 9 bugs that only show on
known matches, including a false MATCH and a mangled permuter target.

**Matching loop order (2026-11-07, built to cut round trips):** stuck on a codegen gap → one
`vsweep.py SRC FUNC pack` (all one-change levers in one call) and `variant.py --proto` for
prototype tests (no hand-made scratch repos); "why does B fix it" → `rtl.py FUNC --diff A.c B.c`
(first pass where they diverge) before more single-guess variants.

**Every new best version gets `vsweep pack` first** (2026-11-17): not only at session start. Each time a lever change makes a new near-match, rerun `vsweep.py SRC FUNC pack` on it before permuter/hand guesses; levers interact, so an earlier pack on another version doesn't cover it.

**Region names and caller scans (2026-11-21):** name a caller region from
`ffx-analysis/data/systemmap.tsv` (or `symuse.py --runs`, which uses it), never by reading the
address (0x21xxxx below 0x21FC28 is battle UI, not menu; I put "menu" into the notes and had to
correct it). Scan callers/refs with `symuse.py` (reads the load image), not by parsing splat asm:
splat emits some code as `.word` + `alabel` (e.g. 0x1AC068-0x1ACEA8 in 171EC0.s), which a
`glabel`/`jal` text parse silently misses.
