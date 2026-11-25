---
name: emulator-scripting
description: Verified PCSX-Redux Lua scripting facts and pitfalls, PCSX2's lack of scripting and the workarounds, and how to hand the user debugger tasks
metadata:
  type: reference
---

**PCSX-Redux Lua (verified 2026-10-27 on FF7, ovltrace):**
- `PCSX.addBreakpoint(addr, 'Exec'|'Read'|'Write', width, cause, fn)` fires on hardware-register writes too. Write breakpoints fire before the store: decode the `sb/sh/sw` at the PC and read its source register for the value.
- `PCSX.getRegisters().GPR.r[n]`, `.GPR.n.ra`, `.pc` work; `PCSX.getMemPtr()` = byte pointer to RAM; `io.open('//wsl.localhost/Ubuntu-22.04/...')` writes straight into WSL.
- **Never return `false` from an invoker AND drop the breakpoint object**: Redux deletes it and the GC presumably deletes it again (crash within seconds). Keep objects for the session; use `:disable()`/`:enable()`.
- `PCSX.GPU.takeScreenShot()` (undocumented) returns `{data, width, height, bpp}`, `tostring(data)` = raw pixels; worked without changing renderer.
- Breakpoints need Debug > Enable Debugger (interpreter).

**PCSX2 (checked 2026-10-27):** no scripting language (maintainer: "We've never had Lua support"). Interactive debugger only (read watchpoints also fire on writes, #12393). PINE = memory read/write + savestates, no breakpoints/events. Workarounds: savestate RAM images matched offline; PCSX2-MCP fork (v2.6.3, JSON over TCP 21512: breakpoints, watchpoints, registers, step, screenshots; pause/poll semantics unconfirmed); or logging added to a custom build. Details: notes Appendix E.

**PCSX2-MCP v1.0.0 access (verified 2026-10-28):** the user runs it on Windows (`D:\Emulation\ps2\PCSX2-MCP-v1.0.0-win64\pcsx2-qt.exe`). The DebugServer binds Windows `127.0.0.1:21512`, so from NAT-mode WSL it's unreachable and no MCP tools appear in Claude Code. Relay through interop: `powershell.exe -NoProfile -ExecutionPolicy Bypass -File $(wslpath -w ~/dev/ffx-analysis/scripts/dbg.ps1) '<json>'`. The protocol is one JSON line per connection: `{"cmd":"status"}`, `{"cmd":"read_memory","address":<int>,"length":N}` (a string address ignores `length` and returns 256 bytes). An unknown cmd returns `available_commands`: status, read_registers, write_register, set_pc, read_memory, write_memory, read_string, disassemble, evaluate, set_breakpoint, remove_breakpoint, list_breakpoints, set_memcheck, remove_memcheck, list_memchecks, pause, resume, step, step_over, get_threads, get_modules, is_valid_address, clear_breakpoints. **No screenshot command in v1.0.0.** Omitted arguments default to 0: a bare `{"cmd":"set_breakpoint"}` set a breakpoint at 0x0, so always pass every argument. With mirrored WSL networking (user enabled it 2026-10-28), connect directly from WSL; client `~/dev/ffx-analysis/scripts/pcsx2.py`. `read_memory` is capped at 0x10000 bytes per call (fast: 2 MB in 0.2 s). `status` returns the PC without pausing (about 1,900 samples/s). **Crash (2026-10-28):** setting a 4th breakpoint while the game ran (`sceCdRead` breakpoint already armed) got no reply, and PCSX2 died. Cause unverified (a set-while-running race in the recompiler, or a hit during the request). Since then: pause before set/remove, resume after, add breakpoints one at a time, and ask the user for a savestate before each test. In `read_registers`, GPR values are 128-bit hex with the low word in the last 8 chars. `Get-NetTCPConnection` via powershell.exe checks whether it's listening.

**Handing the user debugger work:** label every address as data (Read/Write breakpoint + width) or code (Exec); write breakpoints stop before the store; have them paste-check addresses. Propose short test routes from savestates or debug/menu shortcuts, never long story progression (user rejected "new game → world map").
