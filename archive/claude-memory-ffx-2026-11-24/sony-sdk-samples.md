---
name: sony-sdk-samples
description: Sony SDK sample code is a pattern source for FFX lib code (programmers start from it); where the sample trees are and what they cover
metadata:
  type: reference
---

User (2026-11-05): "Programmers are lazy, and example/ref code is often a good starting point which
often doesn't mutate much." Check Sony samples for a lib area's shapes, struct layouts and constants.

- `~/dev/DecompNotes/ref/globe/g20_trunk/ee/sample` (SDK 2.0, ~450 .c): `mpeg/mpegstr` `StrFile`
  (isOnCD / sceCdlFILE fp / fd "for HD stream") = movie_cd.c's CD/HDD split; `ipu/ezmovie` = only
  `sceIpuInit` user (movie_str lead). Later globe trunks have no samples.
- `~/dev/DecompNotes/ref/sce-sdk-242/{sample,docs}` (SDK 2.4.2, Dec 2001, damaged zip: only
  graphics/advanced/deci2/cdvd/basic3d/device samples, 58 English PDFs incl. Movie LibRef,
  movtips, hdd_rule). `cdvd/smp_ee/main.c` = movie_cd.c's sceCdCallback/GetError pattern.
  Full unpack (+ ee9911 toolchain) in `~/dev/ffx-analysis/sdk/x242/`. FFX itself is SDK 2.3.4.
- Files are Shift-JIS: `grep -a`, `iconv -f SHIFT_JIS`. Rank samples by distinctive call sequences,
  not shared-API counts (GS/VU boilerplate dominates).
- Sony headers hold real names for magic numbers (e.g. `SCECdFuncRead 1` from `libcdvd.h`
  "Callback Reason"); check them before inventing a define. Related: [[decomp-reference-techniques]].

- Sample-local types (`sceSamp0IVECTOR`/`FVECTOR` in samples' `mathfunc.h`, `QWdata` in
  `basic3d/core/core.c` and `defines.h`) are copies, not SDK API. Prefer the identical library
  typedef (`sceVu0IVECTOR`, libvu0.h); with no library twin, define it locally in the TU (2026-11-07).
- Give samples by full path: the user couldn't find mpegstr `disp.c` (it is under `ref/globe/...`,
  not `sce-sdk-242` or `ps2sdk`). Other refs: `ref/ps2sdk` (open PS2SDK), `ref/PS2Docs` (manuals).
- Sony SDK headers in globe are under `globe/<trunk>/common/include/` (libcdvd.h etc.) as well as `ee/include/`; check both before saying a constant has no SDK name (SCECdINIT/SCECdCD/SCECdDVD came from common/include).
