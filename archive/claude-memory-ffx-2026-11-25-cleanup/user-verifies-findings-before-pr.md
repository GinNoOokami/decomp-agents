---
name: user-verifies-findings-before-pr
description: "My findings are leads for the user's notes, not PR-ready facts; the user verifies them before using them in code, names or PRs."
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

The user writes up my findings in notes (`~/dev/DecompNotes/<project>/`) and checks them independently before turning them into names, enums or comments in a PR. Don't tell them a name "meets the high-confidence bar" or is ready to submit because I confirmed something. Whether to submit it is their call, once they've verified it.

**Why:** on 2026-10-06 I suggested a counter-mode enum and said the names met the bar "since the materia code confirms it". The user replied: "you confirmed it, not me ... 'My LLM said so' is not a defensible stance in a PR."

**How to apply:**
- Give the evidence (functions, addresses, kernel offsets) so the user can check it themselves.
- Offer names as options to use once they're verified, not as changes to make now.
- Keep cleanup suggestions to the evidence already in their code and headers, and put new findings in the notes doc for later.
- Related: [[naming-high-confidence-only]], [[cite-sources-and-runtime-checks]].

**What counts as confirmed (2026-10-06):** the user only treats evidence in **decompiled, matched C** as confirmed when naming things (enum values, fields, functions). Asm reading, game-data scans (SCENE.BIN, KERNEL.BIN) and my recall are leads, not confirmation. When I list evidence, keep three tiers apart: "in C", "C with one asm link", and "data or hypothesis". Check that each cited function is matched C (not `INCLUDE_ASM` or `NON_MATCHING`) before calling it C evidence.

**Code comments follow the same rule (2026-10-18):** I proposed a `func_800A35F8` comment calling the `D_8016361C` entries "formation IDs" based on my own SCENE.BIN parse. The user rejected it: they can't easily check data files the way I can. Proposed code comments may only state what the matched C shows; data-scan facts go to the notes doc as leads.

**Not a ban on data/wiki facts in comments (user, 2026-10-23):** "you're taking my 'rule' too literally. It's more for you than me, and I'm fine taking wiki or data sources if there is a strong degree of confidence or corroboration." The point is that my unverified claims don't go into code. Wiki or data-scan facts are fine in comments/names when they're strongly corroborated (e.g. wiki table matching the code bit for bit, or checked against KERNEL/SCENE data). Say what corroborates each one; don't refuse or over-hedge corroborated sources.
