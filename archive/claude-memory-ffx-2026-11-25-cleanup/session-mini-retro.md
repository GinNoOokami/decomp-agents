---
name: session-mini-retro
description: "FFX - user wants a short mini-retro at the end of sessions (repeated manual steps, misses, round trips) to iterate on tools and process"
metadata:
  node_type: memory
  type: feedback
  originSessionId: c90454ab-162f-475d-a3c3-7efc470b931a
  modified: 2026-11-20T08:42:31.894Z
---

At session end (after the handoff), offer a mini-retro: repeated manual commands worth a tool, my
process misses and their common cause, avoidable round trips. Keep it short, ranked by cost, with
concrete proposals; tools go through [[tools-lean-evaluate-first]] (offer, build on a yes,
ToolingIdeas.md otherwise).

**Why:** user, 2026-11-20: "I want to get into the habit of doing mini-retros to try and iterate,
perform a self-improvement loop as we work on the project together".

**First retro (2026-11-20), done the same night:** rename.py and swrun.py built (Tools.md), persistent
scratch repo in use ([[persistent-scratch-repo]]), cleanup-applies-trivial adopted ([[cleanup-applies-trivial]]). Original list: rename.py (symbol_addrs + C/headers
rename, refuses on INCLUDE_ASM, adds missing entries, make), swrun.py (PS2 run beside Switch
functions.tsv run with bodies; global users 1:1), persistent rsync'd scratch worktree instead of
cp -a per check, optional "cleanup applies trivial renames" permission change.

**Second retro (2026-11-20 late):** built swfunc `--callers/--callees` and odiff `FFX_REPO`;
retcheck.py (v0-on-return check before writing asm-callee prototypes) went to ToolingIdeas.md;
staging misread fixed in [[cleanup-applies-trivial]] (run `git diff --quiet`).


**Third retro (2026-11-21, tklib wrap-up):** built swfunc `// Alpha` lines; lessons (alpha traces before leads, data exports via symbols.tsv, field greps across src/) in [[switch-counterpart-every-function]].

**Fourth retro (2026-11-21 night, RNG/VU0/core services):** all actions done the same night: [[check-refs-first]] saved; pgrep-before-copy added to [[shell-gotchas]]; scratch.py re-split fix (Tools.md). Miss: wrote "large items align to 16" into notes before compiling (it's `.align 3`); caught and corrected the same pass ([[cite-sources-and-runtime-checks]] applies to notes claims too).

**Fourth retro (2026-11-22, battle RNG file):** misses: renamed by hand with sed 5x although rename.py
exists (check Tools.md before ad-hoc edits); re-raised the known .space warning (memory updated);
claimed "no other RNG" before checking VU0 `vrnext` (scan hardware paths too). Fixed scratch.py --fn
truncate-before-read.

**Fourth retro (2026-11-23, 1ACEA8 stubs + qword copy):** built `variant.py --asm dtls`; `imgdis.py` to
ToolingIdeas.md (symuse doesn't scan instruction patterns). Misses: theorised levers before reading
Toolchain.md's assembler table (lesson in [[ffx-matching-skill]]); answered from the session-start
git snapshot; over-long requested note ([[notes-short-mechanical-facts]]).

**Fourth retro (2026-11-23, battle 1ACEA8):** misses = re-deriving instead of reading the transcript/swrun, hand renames instead of rename.py, staged-WIP misread, two build collisions. Fixes: tools-first hook (see [[tools-first-never-reinvent]]), flock idea in ToolingIdeas.md, memory rules for rating references and name candidates.

**Fifth retro (2026-11-23, MsBattleGetEncountEffectStatus + func_001AE3D0):** misses: hand sed
rename again (hook caught it; rename.py --switch now used); paired Switch neighbours by grepping
functions.tsv/decomp.c instead of swrun.py (hook reminder); hints-only on AE478 took three rounds
before a skeleton helped (offer the skeleton after one failed hint round); confusing match report
([[proposal-vs-applied-wording]]); zsh `rm -f t_*.c` with no match aborted an && chain. Lesson saved:
movz vs cross-jumped if/else in [[ffx-matching-skill]].
