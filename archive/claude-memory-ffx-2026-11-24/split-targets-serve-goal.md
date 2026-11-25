---
name: split-targets-serve-goal
description: "FFX file splits - pick edges that cover functions matched C already calls (extern.h), not the edge with the easiest evidence; scratch-build every proposal; then move prototypes to the area header"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 097ed56a-8827-4283-b2d6-f2206c8d5f47
  modified: 2026-11-10T08:54:06.502Z
---

When proposing file splits, choose the target by the user's goal: files that contain functions
already called from matched C (the declarations in `include/extern.h`), so the split lets those
prototypes move to an area header. Don't pick the edge whose evidence happens to be cleanest.

**Why:** (2026-11-10) I proposed `common/2D8570` because the alpha `tkdbg.c` gap made a neat edge;
it gave the user two functions nobody in C calls ("not super helpful"). They had asked for files
mapping to the extern.h TK functions. The same session the first proposal also failed to link
because I hadn't built it.

**How to apply:**
- Start from extern.h: map each declared function to its cluster, then rank clusters by how many
  declared functions they cover and how cheap their edges are. `symuse.py --fanin --range` for
  spread inside a layer.
- Signals that worked in the TK layer (fast, in order): per-object `.data` blocks (gcc emits one
  block per object, so far-apart blocks = separate files); alpha code International dropped
  (`xmap.py lookup` spacing jumps, usually debug code at a file's end); high `.bss` blocks with no
  outside users (`symuse.py ADDR --size --range`, weak alone).
- Build every proposed yaml/C/symbol change in a fresh scratch copy (`rsync --exclude build
  --exclude .git`, full `make`) before showing it; check each new `.c.o` size in the map
  ([[splat-c-file-gotchas]]).
- After the user splits: move the prototypes from extern.h to the area header (sections per file,
  address order) and add the header include to every C caller ([[header-prototypes-external-only]]).
