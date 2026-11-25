---
name: reusable-outputs
description: "Every result, setup or workflow gets saved in reusable form (tool, note, script) when it is produced; check for an existing one before redoing any work"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 00cc723e-33de-4668-adc1-6155ce71f58b
  modified: 2026-11-04T18:09:04.607Z
---

Don't recreate results or workflows. Anything that produced an outcome is saved in a form that can be
reused, unless it is certainly a one-off: a repeated shell recipe becomes a tool (or an option on an
existing one), an analysis result goes in the notes, and a setup procedure becomes a script.

**Why:** (2026-11-04) the user ran out of weekly tokens the week before. In one session I rebuilt the
permuter dir by hand, rewrote asm-compacting/RTL-cutting one-liners and wrote throwaway diff
scripts (mnemonic diff, variant splice) that already existed or should have been kept.

**How to apply:**
- Before any step, check Tools.md ([[tools-reference]]), the nonmatchings notes
  (`DecompNotes/FFX/nonmatchings/`) and the other notes for an existing tool or result.
- When I write a helper during work, decide at once: keep it (add it to `~/dev/ffx-analysis/scripts`
  or extend an existing tool, list it in Tools.md) or state why it's a one-off.
- Record per-function attempts as they happen: `nmnote.py FUNC` creates the note, `variant.py ... --log --note` appends results; start a function with `facts.py FUNC` (and `--ours`) instead of reading asm by hand.
- See [[token-economy]].
- Persistent notes never point into the scratchpad (session-only): copy what a note references
  next to it (variant.py --log saves definitions in nonmatchings/<ADDR>/).
- Before searching the filesystem for a project resource, grep memory and the notes for it
  (2026-11-04: missed ~/dev/DecompNotes/ref/rozelette-ffx, recorded in two memories).
