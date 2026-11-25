---
name: github-ssh-remotes
description: "The user's own GitHub repos push over SSH (account key); HTTPS has no credential helper on this WSL, so HTTPS remotes can't push"
metadata:
  node_type: memory
  type: reference
  originSessionId: 48b2336b-2a8e-40ab-ad53-efe697ab3ca1
  modified: 2026-10-29T07:18:02.290Z
---

On this WSL install, pushes to the user's GitHub repos (account GinNoOokami) work over **SSH**:
`~/.ssh/config` maps `github.com` to the account key `~/.ssh/id_github_ginnoookami`. No
`credential.helper` exists at any git config level, so an `https://github.com/...` remote reads fine
(public repos) but fails to push ("could not read Username"). Seen 2026-10-29 with the new
`ffx-rearc` repo, created with the HTTPS URL from GitHub's quick-setup page; fix:
`git remote set-url origin git@github.com:GinNoOokami/<repo>.git`.

Networking (mirrored WSL mode for PCSX2-MCP) does not affect GitHub: SSH and HTTPS both connect.

**How to apply:** if a push fails, check `git remote -v` first. See [[emulator-scripting]].
