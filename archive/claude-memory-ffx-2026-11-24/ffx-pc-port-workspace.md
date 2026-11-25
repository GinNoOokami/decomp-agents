---
name: ffx-pc-port-workspace
description: "Steam PC FFX.exe/FFX-2.exe Ghidra workspace, how to re-export, and what PC-port evidence is good for (file names, asserts, logic; not codegen)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 0470921f-48a5-4376-a7a6-302261357fb2
  modified: 2026-11-06T08:46:52.280Z
---

Workspace `~/dev/ffx-analysis/builds/pc/` (FFX) and `builds/pc/FFX-2/`: copied exes, run scripts,
`out/{decomp.c,disasm.txt,functions.tsv,strings.tsv}`. `run_ghidra.sh` = import + analyse
+ export; `export_only.sh` = `-process <exe> -noanalysis` re-export of the saved project (fast). Ghidra 12.1.4 headless projects `FFX` and `FFX-2` live in `~/dev/ffx-analysis/ghidra/pc/`.
Shared export script `builds/pc/scripts/ExportAll.java`. Findings live in DecompNotes
`FFX/PCPortOverview.md` and `FFX/ProjectLayout.md`, not here ([[memory-scope]]).

Gotchas:
- Steamless "fails" on these exes because there is no SteamStub (`.bind`) to strip; analyse as-is.
- A headless `-postScript` Java file that fails to compile only shows as "class could not be
  found" in the log after the whole analysis; check imports (e.g. `ghidra.program.model.symbol.Symbol`)
  before a long run, or re-export with `export_only.sh`.
- `decomp.c` is 37 MB: extract one function with awk on its `// ==== FUN_x @ addr ====` header.
- The export misses functions reachable only through data tables (2026-11-06: 20 of 31 Virtuos
  movie stubs absent, so their strings looked unreferenced). Before calling a PC string or function
  unused, scan raw `FFX.exe` for `PUSH imm32` (0x68) of the string and for pointer words to the
  function (PE sections from the header, imagebase 0x400000). PC `{init,exec,_,result}` script
  tables line up slot-for-slot with the PS2 group tables: a cheap name source.

- Raw disassembly beats the Ghidra export when asked to check the PC code: `objdump -d -M intel
  --no-show-raw-insn -j .text FFX.exe > scratch` takes ~1 s (75 MB), and splitting at `int3` padding
  gives function bodies. Find counterparts by mapping one known global (PS2 struct base + offset ->
  PC absolute address) and grepping it (2026-11-23, func_001AF188 = PC 0x782AE0).

What PC evidence is good for: logic, constants, struct shapes (port added fields), original
`__FILE__` paths (folder grouping) and assert expressions. Not codegen (MSVC 2012 /O2), not
function order inside a file. Before using a PC assert, check the PS2 ELF's own `func_00157788`
asserts: they usually carry the same expression text, which is direct evidence ([[cross-version-analysis]],
[[decomp-reference-techniques]]).

**Use the PC version for control flow when m2c's output is goto soup** (2026-11-02): the PC
decompile of func_00163450 (`FUN_0076d200`, debug printfs survive there) and func_00163768
(`FUN_0076d440`) compiled to the PS2 shape almost directly, with plain loops and `break`/`continue`.
Find the counterpart by grepping `out/decomp.c` for a unique string or constant (assert text,
`"DVD FILE"`, `0x1800`) and keeping hits in the same address area (libmscd ≈ 0x76B000–0x770000),
or by looking at a known counterpart's callees. Its Ghidra `do { } while` + `if (n != 0)` is just a
`for` loop. Expression shapes (pointer vs index) still need PS2 testing.
