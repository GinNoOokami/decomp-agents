---
name: community-tools-corroborate-only
description: "Community/modding tools (e.g. FFXDataParser) are corroborating data points, never a source of truth for names or structure"
metadata:
  node_type: memory
  type: feedback
  originSessionId: e7f4ef19-a295-4d48-81be-17d03ea74cdb
  modified: 2026-10-28T22:42:50.870Z
---

Use community and modding tools (e.g. Karifean/FFXDataParser's script function names) as corroboration only. A name or meaning from them counts only where our own evidence (code, data refs, runtime) agrees, and the note cites both.

**Why:** 2026-10-28 the user said "Let's use it as a corroborative data point, not a source of truth" after we found FFXDataParser's group names line up with the ELF's script tables. Those names come from script behaviour and are often marked `?`, which is weaker than decomp-derived evidence.

Same for other decomp projects of the game (Rozelette's FFX): their names corroborate, and the user doesn't want to redo files they already decompiled (`mem.c`, `mem2.c`, `elf.c`); pick new files instead (2026-10-29).

**How to apply:** label tool-derived names as outside/unverified until our evidence matches; never name a function from the tool alone. Record the tool's commit hash. Related: [[state-evidence-at-real-strength]], [[goal-quality-reconstruction]].

**Look there first for unknown callees (2026-10-30):** before working out what an unnamed callee does from asm, grep `~/dev/ffx-analysis/ref/symbol_addrs.txt` (Rozelette's names: e.g. `Mem_Alloc` 0x11D370, `sceOpen`/`sceLseek`/`sceClose` 0x2FC1A0/0x2FC5A8/0x2FC428, `memset` 0x2EED60) and their matched C in `~/dev/DecompNotes/ref/rozelette-ffx/src/`. I spent a round decoding `Mem_Alloc` from asm that their files already had. Quote it as a lead, then check it against our evidence.

**Matching stalls: grep the ref decomps before any sweep (2026-11-09):** when a match stalls, the first step is to grep `~/dev/DecompNotes/ref/khasinski-ffx` and `rozelette-ffx` for the function's globals or strings. Their C can be tested in scratch as a variant in seconds. On `func_0011F8F8` I spent most of a session on type/order/struct sweeps and the PC port; khasinski-ffx's statement order matched at once, and the user called it a big miss. A working order from them is a lead to verify with variant.py, not a reason to skip our own evidence.
