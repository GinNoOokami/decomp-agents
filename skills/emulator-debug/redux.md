# PCSX-Redux (FF7)

Verified 2026-10-27 on FF7 (ovltrace). Full write-up: `~/dev/DecompNotes/StartingADecompProject.md`
Appendix E.

- Breakpoints need Debug > Enable Debugger (interpreter).
- `PCSX.addBreakpoint(addr, 'Exec'|'Read'|'Write', width, cause, fn)`; write breakpoints fire before
  the store (decode the `sb/sh/sw` at the PC and read its source register for the value) and also
  fire on hardware-register writes.
- `PCSX.getRegisters().GPR.r[n]`, `.GPR.n.ra`, `.pc`; `PCSX.getMemPtr()` = byte pointer to RAM;
  `io.open('//wsl.localhost/Ubuntu-22.04/...')` writes straight into WSL.
- **Breakpoint lifetime:** never return `false` from an invoker *and* drop the breakpoint object
  (Redux deletes it, the GC deletes it again: crash within seconds). Keep the objects for the
  session; use `:disable()`/`:enable()`.
- `PCSX.GPU.takeScreenShot()` (undocumented) returns `{data, width, height, bpp}`.
