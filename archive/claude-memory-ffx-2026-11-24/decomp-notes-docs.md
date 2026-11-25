---
name: decomp-notes-docs
description: "The user's research notes live in ~/dev/DecompNotes (git repo, commit+push after every edit); write-ups go there as .md, FFX work in FFX/"
metadata:
  node_type: memory
  type: reference
  originSessionId: ea1f2dae-87a9-458f-821f-a00942b27c5d
  modified: 2026-11-08T20:48:06.721Z
---

`~/dev/DecompNotes/` holds the user's markdown research notes for all decomp projects (copied from the ff7-decomp memory, 2026-10-28).

**Layout:** `FFX/` (this project: `FFX/OverlayIdentification.md` is the starting handoff), `FF7/`, `PSX/` (PSX toolchain), root = general guides (`StartingADecompProject.md`: appendices C = PS2/FFX, E = ovltrace runtime tracing + PCSX2 options, F = ff9-decomp techniques, B = MSVC PC), `tools/` = reusable scripts (`tools/ovltrace/`), gitignored `ref/`.

**Index:** one `README.md` per project (`FFX/`, `FF7/`, `PSX/`), one line per doc; the root `README.md` only links them (user, 2026-11-08: "Shouldn't each project have its own index?"). Read the project's index first to pick which notes to open instead of grepping blind; add a line whenever a doc is created (same commit). `FFX/nonmatchings/README.md` stays its own index.

**Git:** repo `git@github.com:GinNoOokami/DecompNotes.git`, branch `main`, pushes with a deploy key via local `core.sshCommand`. After every note edit, commit and push right away without asking: `git -C /home/jeff/dev/DecompNotes add <files>`, `... commit -m "<msg>"`, `... push` (plain push only, never force). Brief, specific messages naming the note and change; no Co-Authored-By or other attribution. Check `status` first and stage only files I edited (the user edits notes too).

**Rules:** "write it up / make a doc" = a readable .md here (PascalCase name), never a claude.ai doc. Read the relevant note before re-deriving something. Edit notes only when asked; read before editing. Label each claim as code-verified, from community docs, or a hypothesis. The notes are the user's own: no project CONTRIBUTING/LLM-stance rules apply to them.

**Troubleshooting.md = problems hit and fixed in the repo only** (user, 2026-11-04: "No, no
troubleshooting entry" for an untested/held build change); candidates and held proposals go in the
topic note (e.g. Toolchain.md) labelled as candidates.
