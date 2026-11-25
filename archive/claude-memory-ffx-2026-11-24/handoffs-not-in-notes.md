---
name: handoffs-not-in-notes
description: "Session handoffs go in the git-ignored DecompNotes/handoff/ folder, never in tracked notes; notes hold only facts, evidence and results"
metadata:
  node_type: memory
  type: feedback
  originSessionId: e7f4ef19-a295-4d48-81be-17d03ea74cdb
  modified: 2026-10-29T01:10:40.291Z
---

Write session handoffs (status, next steps, "resume here") to `~/dev/DecompNotes/handoff/<project>.md`, which is git-ignored, or as text to paste into the next session. Don't put handoff, resume or next-session sections in tracked notes or the log.

**Why:** 2026-10-28 the user said "I don't think there's any value in encoding these in the commit history, only facts, evidence, and results."

**Always update the handoff at the end of every session** without being asked, unless the user says not to or says to skip it at the start (user, 2026-11-07: "always update the handoff at the end of a session"). Wrap-up = learnings to memory + handoff.

**How to apply:** at session end, overwrite the handoff file and mention its path; tracked notes and `Log.md` get only findings, evidence and reference info (tools, data, scripts). Related: [[keep-working-log]], [[decomp-notes-docs]].

**Check every open item against the current tree before writing it** (2026-11-02: I listed
func_00163768's `tailSize` naming as open, but the user had already renamed it to `headSize`).
The user edits and commits between messages, so an item from earlier in the session may already
be done: grep the source or `git log` for each one first ([[verify-match-status-from-source]]).

**Copy the handoff to the session scratchpad at session start** (2026-11-15: another agent rewrote
`handoff/FFX.md` mid-session; the user asked whether I had a backup, and only my session-start
read in context saved it). The folder is git-ignored, so there is no history to restore from.

**Rewrite in place, open items only (2026-11-21):** the file had grown to 424 lines / 28.5 KB of
five stacked handoffs with "carried over unchanged" sections. User: "Done items should absolutely
not be kept, same as notes"; they were considering a hard size limit. So: one handoff, rewritten each
session (Current tree / Next / Open items / Parked), no "Done this session", no carried-over copies,
one line per item with a pointer to the note that holds details; preferences live in memory, not
there. Rewritten to 55 lines / 3.9 KB; keep it well under ~100 lines.
