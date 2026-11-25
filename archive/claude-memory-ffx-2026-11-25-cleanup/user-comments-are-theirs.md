---
name: user-comments-are-theirs
description: "In cleanup, never strip or reword the user's own comments or revert their style choices (e.g. decimal constants); the comment rules govern my comments only"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

The comment rules (no matching-shape comments, only facts evident from the C, see [[cleanup-checklist]] and [[user-verifies-findings-before-pr]]) apply to comments **I** write. For the user they're only guidelines. In a cleanup pass, keep the user's comments and their deliberate style choices as they are. Don't revert their changes to mine.

**Why:** on 2026-10-19, cleaning up func_800ACE88, I removed their dead-code comments as "matching-shape / not C-evident" and turned their decimal `252` back into `0xFC`. They found decimal more readable. Their comments explain why unused code is there at all, and that isn't obvious from the code itself, since part of it isn't even in the asm.

**How to apply:** build on the user's last pass. Keep their number formatting, and list only the additions I make. The user said I'm welcome to suggest improvements to their comments. Outright false or misleading comments I may change directly; say what changed and why. Otherwise leave their comments as they are.

**Reviews too (2026-10-21):** when reviewing the user's changes, don't flag their comments for breaking my comment rules (e.g. a hedged supposition from scene data, "probably used in the Bizarro-Sephiroth fight"). Suppositions that are the only thing that fits are deliberate hints to future readers. Flag only comments that are factually wrong (e.g. naming a function that doesn't exist on the branch).

**Transcription notes (2026-11-03):** short `// getFoo?`-style comments in the user's C are their own guesses from transcribing m2c, not recovered names or evidence; never cite them as evidence. Replacing one with a proper description during a cleanup is fine.
