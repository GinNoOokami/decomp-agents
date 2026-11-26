# decomp-agents

Shared knowledge for the coding agents (Claude Code, Codex) working on my decomp projects. Private.

## Layout

| Path | What |
|---|---|
| `skills/<name>/` | Agent skills: `SKILL.md` (frontmatter `name` + `description` only) plus reference files |
| `archive/` | Snapshots of agent memory before content moved here; local history, never copied elsewhere |

Skills: `ffx-matching` (FFX matching: working agreement, procedure, ee-gcc lessons by symptom),
`ffx-cleanup` (FFX cleanup of matched code: checklist, apply-vs-propose, names, comments, constants,
declarations).
`emulator-debug` (runtime checks: when to test, recipes and what they prove, autonomy, reporting;
per-emulator files for PCSX2-MCP and PCSX-Redux).

## How agents load it

Each skill folder is symlinked into both agents' user skill directories:

```
ln -s ~/dev/decomp-agents/skills/<name> ~/.claude/skills/<name>
ln -s ~/dev/decomp-agents/skills/<name> ~/.codex/skills/<name>
```

New sessions pick up changes; a running session keeps what it loaded at start.

## Writing rules

- Agent-neutral wording: no tool names specific to one agent, no frontmatter fields only one agent reads.
- One home per fact: game facts stay in DecompNotes, recurring problems in `FFX/Troubleshooting.md`
  (cited, not copied); skills hold working knowledge.
- Lessons name the function that verified them.
- Pushes use this repo's deploy key (`core.sshCommand`), not the account key.
