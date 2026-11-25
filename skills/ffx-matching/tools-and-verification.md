# Tools and verification

Locations and options: `~/dev/DecompNotes/FFX/Tools.md`. Check it before writing any script.

## Compiling and comparing

- **Every scratch compile of a candidate goes through `variant.py FILE FUNC -f snippet.c`**; pass
  `--repo SCRATCH` when headers change (without it, moved prototypes become implicit declarations).
  No hand-made compile/compare scripts.
- **variant.py traps:** `-e` replaces the first occurrence in the whole file, so anchor on text unique
  to the function; `-e` on an INCLUDE_ASM line fails ("edit not found"), use `-f`; `-f` needs an
  existing definition (stub the INCLUDE_ASM on a scratch copy); a conflicting header prototype can be
  sidestepped with `#define func_X func_X_hdr` around the include; `--proto` also rewrites the file's
  own declaration.
- **Full build** for "matches": `scratch.py` syncs the scratch repo, applies an edits file and runs
  `--make`. variant.py masks relocated fields and jal targets, then on a match checks each named
  relocation by address (swapped globals fail); section-relative relocations (string literals, jump
  tables) and unresolved names stay unchecked, and data layout isn't compared.
- **Compare assembled objects, never `-S` text.** Most branches are emitted under `.set reorder`; the
  line after a branch is not its delay slot there and gas inserts the `nop` (func_001648F8: a false
  "unsolved difference").
- **Keep branch targets when comparing:** masking branch offsets hid a `break` vs `return` difference
  (func_00165B18, reported as equivalent). Mask only relocated immediates.
- **Check scratch inputs compile cleanly** before analysing their RTL (func_002D1EA0: a "makes pointer
  from integer" warning meant a 1-D `u8` array index was a byte load, and the whole comparison ran on
  wrong code). Fix what the target plainly shows first (advancing pointer, every store present).

## Near-match levers

- `vsweep.py SRC FUNC pack`: one-change levers in one call (`sign:` local signedness, `ret:` callee
  return type incl. missing prototypes, `volatile:` globals, `inline:` single-use temps, `self=s32`).
  Rerun on every new best version.
- `vsweep.py orders` / `moves` / `decls`: statement-order, single-move and declaration-order sweeps;
  report coverage.
- `variant.py --proto NAME=...`: prototype tests; compare with and without.
- `rtl.py FUNC --diff A.c B.c`: first pass where two versions diverge. `rtl.py --priority`: global
  allocation priorities.

## Probing gcc directly

- Compile with `-dS`/`-dR`/`-dL`/`-da` in a scratch dir and read where an insn sits after a pass;
  faster than scoring whole variants. rtl.py's cut dumps drop loop.c's report: read raw `.loop`.
- Pass sources: `cse.c`, `gcse.c`, `loop.c`, `jump.c`, `reorg.c`, `haifa-sched.c` (`sched.c` is
  unused), `local-alloc.c`, `global.c`, `alias.c`, `expr.c`, `stmt.c`, `fold-const.c`.
- Probe the exact shape (void vs value, signed vs unsigned) before declaring something "not gcc".

## Permuter (last ditch)

- Only once control flow, types, layout and statement shape are believed right and a small codegen gap
  remains; say why structure is settled before starting it.
- `~/dev/ffx-analysis/scripts/perm.py go <func> [--base f.c]`, then `watch/status/best/stop`. Never set
  it up by hand: perm.py handles context, inline bodies and the target wrapping (`.set noat` /
  `.set noreorder` like INCLUDE_ASM; without it gas refills delay slots and the target is mangled).
- Differing relocation symbol names cost points (a literal 0 where the `.s` has a D_ label never scores
  0).
- Its outputs are hints to explain: improvements are often semantic changes or noise (a `(float)`
  cast; a fake `do {} while (0)` when run on a version whose loop structure was still wrong).

## Tool hygiene

- After changing any per-function tool (ffxfunc, variant, facts, perm, dme, rtl, vsweep), run
  `toolcheck.py run`; all known matches must pass (its first run found 9 bugs, including a false MATCH
  and a mangled permuter target).
- Per-TU helpers (`tidy_externs.py`, `decompile.py`) run on scratch copies only (`--repo`): some
  write their input, and changes to them get tested there before the user runs them.
- Never leave dumps or temp files in the tree (`gcc -da` writes to cwd).
