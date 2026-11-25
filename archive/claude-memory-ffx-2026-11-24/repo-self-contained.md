---
name: repo-self-contained
description: "FFX repo comments must not reference DecompNotes; code is the documentation (types/names first), code comments next, notes only for investigations the code can't show"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 48b2336b-2a8e-40ab-ad53-efe697ab3ca1
  modified: 2026-10-29T05:17:30.320Z
---

In `~/dev/ffx` (the user's own decomp repo), comments and docs must not point at DecompNotes, step
numbers, or other private material. Put the evidence itself in the comment ("called by entry; calls
Mem_Init, then loops on the frame function") instead of "see SystemSplit step 10".

**Why:** the repo is private for now but may go public; references to private notes would be useless
to anyone else.

**How to apply:** when drafting files for the repo, write self-contained comments. Notes can still
cite the repo.

Findings pertinent to specific code (what a function does, hand-written asm evidence, why a
return shape matches) belong as comments in the code, not in DecompNotes (user, 2026-10-31). When
offering a write-up for such a finding, propose a code comment; notes are for cross-cutting
findings (system split, names, toolchain). See [[ffx-start-here]], [[small-files-inline]].

**Code is the documentation (user, 2026-11-04):** prefer making the code self-evident (types, names,
struct fields) over writing anything down. Add to external docs only for active investigations
whose result the code can't show. Don't offer note updates for findings that a type or name in
the code will carry (e.g. "D_0057C71C is the current SaTask" → type it, don't write it up).

**Prune as code solidifies (user, 2026-11-05):** investigation notes (e.g. FFX/SaTask.md) list only
open items; when a match, type or comment in the repo settles one, delete it from the notes in the
same step (and offer that deletion). Notes shouldn't grow endlessly.

**Leads and facts live in the code (user, 2026-11-15):** name leads for globals/functions go as
comments in the code (e.g. `extern s32 D_0078B708; // port of the last issued command, no Switch
name`), not in notes or symbol_addrs. Notes = investigations, unknowns, tracking, system overviews.
Don't offer to mirror code leads into notes.
No `Lead:` prefix on such comments (user, 2026-11-15); state the candidate and why it's uncertain.
