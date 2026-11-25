---
name: cleanup-delivery-format
description: "Deliver cleanups as an in-chat diff or the whole function (file or chat), never as a .patch file"
metadata:
  type: memory
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

Don't hand over cleanups (or other multi-file proposals) as a patch file. Show a diff in the chat, or put the whole cleaned function in a scratchpad file with one `cat ... | clip.exe` line (in the chat if short).

**Why:** the user (2026-10-21, BATTLE_Main cleanup) never applies a patch wholesale, and a unified-diff file is hard to read visually.

**How to apply:** function body → whole function in a file, see [[long-text-to-file]]. Header/config/rename changes → short per-file diffs in the chat. Part of [[cleanup-checklist]] delivery.
