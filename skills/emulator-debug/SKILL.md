---
name: emulator-debug
description: Runtime checks in an emulator debugger for the decomp projects (FFX on PCSX2-MCP, FF7 on PCSX-Redux) - when a runtime test is worth it, setup and connection checks, recipes (who writes an address, hold or poke a value, breakpoint logging, PC sampling), what each proves, autonomy rules, reporting and cleanup, plus each emulator's API quirks. Use when verifying a field's writers or meaning at runtime, logging calls or loads, or handing the user a debugger task.
---

# Emulator debugging

A runtime check answers what the static pass can't settle: who writes a value in play, what breaks
when it is held, which calls a game action makes. It costs the user's time (booting, playing to
the right spot), so it comes after the static pass and starts with a written prediction.

Per-emulator files:

| Emulator (project) | File |
|---|---|
| PCSX2-MCP (FFX, PS2) | pcsx2.md |
| PCSX-Redux (FF7, PS1) | redux.md |

## Working agreement

- **Static pass first.** Before proposing a run: search *all* of the disassembly for the address
  and its struct offset (`0xOFF(` stores, the big unsplit `.s` files too), follow copies and
  pointers, read the counterparts. Write the prediction down ("MsBattleExe clears it, the state
  machine sets it at status 10"); the run confirms or refutes it.
- **Honest cost.** Say what the user has to do (boot, savestate, which game state) and how rare
  that state is. Prefer savestates and common states (a random battle, a menu); never long story
  progression.
- **Autonomy once the user has the emulator running:**
  - read-only steps run directly: connection checks, memory reads, memchecks/breakpoints that
    only log, PC sampling;
  - anything that changes game state is announced first, in one line: memory writes (pokes),
    holding a value, PC changes, stepping past code.
- **Savestate before each test** (the user makes it); a crash then costs nothing.
- **Clean up every run:** memchecks and breakpoints removed (check both lists; pausing and
  stepping can leave temporary breakpoints), background pollers stopped, the game resumed. Say so
  in the report.
- **Background, never blocking:** long runs go in the background; the user plays and says when the
  game state is reached; read the log then.

## Procedure

1. **Connect and confirm the build:** the status call answers; read two known instruction words
   (e.g. the stores the prediction names) and compare with the disassembly. A mismatch = wrong
   build or region.
2. **Arm the minimum:** one memcheck or breakpoint at a time, set while paused (emulator file).
3. **Let the user play** to the state; log in the background.
4. **Read the log, then extend:** a refuted prediction means a writer was missed; search again
   statically before arming more.
5. **Manipulate only to test meaning:** hold or poke after the writers are known, with the
   predicted visible effect stated first ("held at 0, the transition should never finish").
   Tell the user exactly which inputs to try and what each should do (a poke changes nothing
   until the user acts: "press Up/Down and Left/Right on the target cursor").
6. **Report and clean up.**

## Recipes: what each proves

| Question | Recipe | Proves | Doesn't prove |
|---|---|---|---|
| Who writes X? | write memcheck, log each pause (PC, instruction, value, game state), resume | every CPU write in the played route | writes by DMA or in routes not played; writes before arming |
| What does X do? | hold: step past the store, write the old value back; or poke a value | X's effect in that state (black world view while held) | which reader caused it (read the code) |
| Does X stay set? | read X after the state ends (back on the field) | its lifetime | why nothing clears it |
| Which calls/loads happen during an action? | pausing exec breakpoints on dispatchers, a label file the user's narration updates | the calls in that run | completeness (only the played route) |
| Where does the game spend time? | PC sampling through the non-pausing status call | hot code per mode | call order |
| Is RAM code the ELF's? | dump `.text` and compare words | runtime patches (FFX: `printf` detoured to a no-op) | when they were made |

Traps for all recipes:
- A write memcheck pauses **before** the store; the store may be in the delay slot of the paused
  PC. Holding needs the PC of the store itself.
- Breakpoint hit rate slows the game: FFX's script dispatchers at 122 hits/s made it "near
  unplayable", 30/s was barely noticeable. Pick the lowest-rate hook that answers the question.
- User narration lags: an event logged under a label usually belongs to the end of the label
  before it.
- Absence in one run is weak evidence; say which route was played.

## Reporting

- A table: time, PC, instruction, value, game state (status bytes read with each pause).
- Observations from the run are stated plainly in code comments ("not cleared when it ends");
  inferences keep `?`. Notes get only cross-cutting findings.
- Name what the run did not show (the moment a transition resumed, unplayed routes).
