---
name: struct-offset-tags
description: FFX - every struct member I propose gets a /* 0xNN */ offset tag; a style guide is planned (not now)
metadata:
  node_type: memory
  type: feedback
  originSessionId: c90454ab-162f-475d-a3c3-7efc470b931a
  modified: 2026-11-20T02:01:32.773Z
---

Every struct I propose or edit in ~/dev/ffx gets `/* 0xNN */` offset tags on each member
(`/* 0x0E */ u8 unkE[0xC2 - 0xE];`), like movie.h / libmscd.h / sa_task.h / tklib.h.

**Why:** 2026-11-19 I proposed `Unk00319150` untagged; addfields/structfields then couldn't read it
(preview diff dropped the fields). The tool now infers offsets for untagged basic-type members, but
the user asked for tags going forward. User: "Probably worth making a styleguide for this kind of
thing, but not right now" - offer it later, don't start it unasked.

**How to apply:** tag in every struct proposal, including gap arrays; related [[tools-reference]],
[[no-layout-nits-clang-format]].

**No `size:` tag on structs ending in a zero-length array** (2026-11-24, user): `sizeof` is only the
header, so the tag misleads; the `[0]` member already says the length varies.
