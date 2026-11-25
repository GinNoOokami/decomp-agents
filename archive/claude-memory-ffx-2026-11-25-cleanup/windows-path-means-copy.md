---
name: windows-path-means-copy
description: "When the user gives a /mnt (Windows) path to a file, that is permission to copy that file into WSL"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 8c5ad48d-8d16-4ca4-846e-224a39661a2e
  modified: 2026-10-28T07:51:16.763Z
---

If the user names a specific file on a Windows drive (e.g. `D:\...\game.iso`), copy it into the WSL work folder straight away; don't ask again.

**Why:** The user said "giving you the path was implicit permission to copy it" after I asked for confirmation.
**How to apply:** The permission covers only the file named. Searching or listing other things on /mnt still needs asking (global CLAUDE.md).
