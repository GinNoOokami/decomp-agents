# PCSX2-MCP (FFX)

## Setup and connection

- The user runs the PCSX2-MCP fork (v1.0.0) on Windows. Its DebugServer listens on
  `127.0.0.1:21512` **only once a game is booted and running**: at the game list a TCP connect
  hangs. WSL reaches it only with mirrored networking (`networkingMode=mirrored`); before that,
  `scripts/dbg.ps1` relayed through `powershell.exe`.
- Client: `~/dev/ffx-analysis/scripts/pcsx2.py` (`req`, `status`, `read`, `write`). One JSON line
  per connection.
- Build check: read a known instruction word, e.g. `pcsx2.read(0x1AE078, 4)` = `fc2000a2`
  (International `MsBattleExe` store).

## Server API facts (verified)

- **Omitted arguments default to 0.** A bare `set_breakpoint` set one at 0x0; a wrong `set_pc`
  argument would jump to 0. Always pass every argument; probe an unknown argument name only
  with commands where a wrong name is harmless (`write_memory` with a wrong key writes 0 bytes).
- `set_memcheck {address, size, type: 'write'}`: size 1 is widened to a 4-byte range (log
  `bytes` to tell which byte was hit). `remove_memcheck {address}`. `list_memchecks` reports
  `hits`/`last_pc`/`last_addr`, which stayed 0 after real hits: never read them.
- A memcheck hit **pauses** the game before the store; poll `status` for `paused`.
- `write_memory {address, data: '<hex>'}` returns `written`; check it equals the length.
- `step` returns `old_pc`/`new_pc`. `disassemble {address, count}`.
- `read_memory` caps at 0x10000 bytes per call (`pcsx2.read` loops). `read_registers`: GPRs are
  128-bit hex, low word in the last 8 characters.
- `status` doesn't pause (~1,900 calls/s: usable for PC sampling). `status.cycles` is 32-bit and
  wraps every ~15 s at 294.912 MHz: a lower value is not a savestate load.
- No screenshot command.
- **Crash:** setting a breakpoint while the game ran (several already armed) got no reply and
  PCSX2 exited (cause unverified). Pause before every set/remove, resume after, one at a time.
- The kernel idle loop is around PC 0x81FC0; a pause there is not a hit.
- `pcsx2watch.py` runs (log and hold) left **temporary stepping breakpoints** at pause/step PCs
  (2026-11-25: 0x1AFAEC, 0x1E7320) after removing their memcheck. Cleanup = `list_memchecks`
  **and** `list_breakpoints` (`req({'cmd': ...})`), remove leftovers while paused.

## Tools (`~/dev/ffx-analysis/scripts/`)

| Tool | Use |
|---|---|
| `pcsx2watch.py ADDR [--show NAME=ADDR[:LEN]] [--hold HEX --at PC] [--minutes N] [--log F]` | who writes an address; hold a value past one store; always removes its memcheck (also on SIGTERM) |
| `fnlog.py <secs> exec` | script native-function calls via the dispatchers (a0 = id); label file `out/fnlog.label`, stop file `out/fnlog.stop` |
| `loadlog.py` | archive loads / CD reads / effect loads via breakpoints |
| `pcsample.py`, `modecover.py` | PC sampling by region; hottest functions per mode |
| `slotcheck.py` | compare a loaded effect slot (0x01B00000) with a module file |

## FFX runtime facts used as checks

- RAM `.text` matches the ELF except a `printf` detour to a no-op (0x2EEED0 → `jr ra` at
  0x165AE0, displaced code at 0x1715A0).
- Battle state `btl` = 0x316F70: status byte `+0x10` (0x316F80, 0 = no battle, 0x0A while the
  battle scene starts, 1 in battle), encounter effect type `+0x105` (0x317075).
- Effect modules load at 0x01B00000.

## Script-set values revert pokes

A field a script command writes is often re-asserted (2026-11-25: battle command 0x7099 rewrote
`btl` +0x20FD every cursor move, so a poke lasted 1-3 s and looked like "no effect"). Before
reading a poke as refuted, check the write log for a writer after the poke; then use `--hold`.

## Worked example (2026-11-25, `btl.draw` +0x20FC)

Prediction from asm: `MsBattleExe` clears it, the state machine sets it at status 0x0A. Logging
run: exactly those two writers, and it stays 1 after the battle. Hold run (`--hold 00 --at
0x1AFAE8`): the battle ran with UI but a black world view (effect type 0 transition waiting);
poking 1 restored the scene. Result: the field comment states the observed role.
