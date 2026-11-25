---
name: lead-comment-shape
description: "FFX lead comments on fields/functions - one short line stating what it probably is, with \"?\"; no evidence essays, no provenance restating the code"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 95f31716-f6f0-456e-9d01-695ebd63b267
  modified: 2026-11-17T00:12:34.592Z
---

A lead comment is one short line saying what the thing probably is, marked with `?`. Example the user
wrote for the pad struct's +0x80 byte: `u8 id; // the controller type ID libpad reports?`

**Why:** 2026-11-16 I first wrote a three-line comment with asm evidence and protocol details ("way
too long, let's stick to what we know in matched C"), then cut it to `// returned by TkGetControlerId`,
which only restated the code ("even less helpful"). The user wants the hypothesis itself, briefly.

**How to apply:** the evidence goes in chat (or notes for investigations), not the comment. Don't
restate who reads/writes the field; the code shows that. Related: [[repo-self-contained]],
[[line-comments]], [[user-comments-are-theirs]].
