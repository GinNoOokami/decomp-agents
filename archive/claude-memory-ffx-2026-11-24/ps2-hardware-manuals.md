---
name: ps2-hardware-manuals
description: Sony PS2 manuals in DecompNotes/ref/PS2Docs, greppable text copies in ref/PS2Docs-txt; how to get PDF page numbers
metadata:
  type: reference
---

Official SCE manuals (EE/GS/VU/SPU2 user manuals, BOOKMARKS.md index) are PDFs in
`~/dev/DecompNotes/ref/PS2Docs` (user added 2026-11-07). Text copies made with
`pdftotext -layout` (poppler-utils, installed) are in `~/dev/DecompNotes/ref/PS2Docs-txt`
(git-ignored like all of ref/). PDF page = number of form feeds before the line + 1 (matches the
viewer's page numbers; GS XYZF2 = p139 agrees with BOOKMARKS.md). Skip the table-of-contents hits
(pages ≤ ~12) when looking for a section.

Pages used so far: EE manual DMA source chain 45-48, 58-59; GIFtag 151; PACKED/A+D 152-156.
GS manual PRIM 116, RGBAQ 119, TEX0 125, TEX1 127, TEXFLUSH 131, UV 135, XYZF2 139.
Related: [[user-ps2-graphics-background]], [[sony-sdk-samples]].
