---
name: shell-gotchas
description: "zsh/grep/awk traps hit in this environment and their fixes"
metadata:
  type: reference
---

- zsh doesn't word-split unquoted `$VAR`: use `${=VAR}` or an array `(${(f)"$(cmd)"})`.
- Sony SDK headers are Shift-JIS/CRLF: grep may treat them as binary and print nothing; use `grep -a` and `tr -d '\r' | iconv -f SHIFT_JIS -t UTF-8 -c`.
- After renames, awk scripts that track the current function by `glabel func_` misattribute code to the previous function; match any `glabel`.
- `make` scratch checks: Makefile `VENV_BIN` = `.venv/bin/` (clang-format lives there).
- zsh `=word` expansion: an unquoted word starting with `=` (`echo ======` as a separator) fails with
  "= not found" and aborts the rest of the line. Quote it or use `echo ---`.
- A quoted flag+value in a loop (`for t in "" "-I $dir"`) reaches gcc as one argument `-I /path`
  with a leading space, so the include dir is silently ignored. Use `${=t}` or separate words.
- Python one-liner trap: `open(p,'w').write(f(open(p).read()))` opens (truncates) for writing
  before the read runs, so every file comes out empty. Read into a variable first, then write
  (cost a scratch copy, 2026-11-05).

- **FFX image offsets (2026-11-07):** the flat load image `~/dev/ffx/build/jp_intl/orig/SLPS_250.88.bin`
  starts at address 0x100000 (`xxd -s $((addr-0x100000))`); the disc ELF (`orig/jp_intl/SLPS_250.88`,
  `iso/`) has a 0x1000-byte header first (LOAD offset 0x1000), so its offset is `addr-0x100000+0x1000`.
  Give absolute paths: from ffx-analysis the relative `build/...` path doesn't exist (the user took
  that for a deleted file and switched to the ELF).
- **symuse.py's default range runs to the next symbol**: a string symbol at the start of a block
  (D_0056B3B0: 0x1A0 bytes) "shares" every other string in it. Use `--size` (strings: `--size 1`)
  before calling a string or small object shared.
- **objdump folds zero words**: `objdump -d` prints a run of `nop`s as `...`, so word-by-word
  compares and side-by-side pastes go wrong (looked like missing loop padding, 2026-11-11). Use
  `objdump -d -z`.

- **`pkill -f PATTERN` kills its own shell** when PATTERN appears in the command line (2026-11-17: `pkill -f permuter.py` inside a longer command exited 144 and nothing after it ran). Use `pkill -f "[p]ermuter.py"` or run it as a separate command. Same for `pgrep -f` in wait loops: `while pgrep -f 'make x'` matches its own shell and never ends (2026-11-22); use `pgrep -f '[m]ake x'` in its own Bash call: the bracket trick fails when the same command line also contains a plain `make` (2026-11-22 false hit).

**Concurrent builds (2026-11-19):** the user sometimes runs `make` in ~/dev/ffx while I do. Symptoms:
"Can't open asm/... .s", undefined `D_...` refs at link, then the next `make` passes. Rerun once before
treating it as a tool/splat problem; don't propose Troubleshooting entries for it.

**Never run make alongside the user's build (2026-11-20):** my rename.py make and their
`make format build` both ran splat; the link failed with hundreds of undefined D_ symbols
(half-written undefined_syms_auto.txt). `pgrep -af 'make|splat'` before any make in ~/dev/ffx, and before copying files into the tree (2026-11-21: copied mid-build); if
busy, wait for that PID in the background, then rebuild.


**Quote heredocs that write code (2026-11-21; repeated 2026-11-22: unquoted `python3 - <<EOF` to interpolate a path split every asm `\n` in an applied function; pass paths as argv with `<<'EOF'`, and `grep -c '^"$'` the result):** `cat > f.py <<EOF` (unquoted) turns `\\n` into
`\n`, which Python then reads as a real newline inside a string literal (SyntaxError); it broke two
edits files in one session. Use `<<'EOF'` for any file holding escapes, and put shell values in via
a separate variable or a quoted path, not inside the heredoc. Never hand-edit a unified diff's
hunk body (counts break, `patch` reports "malformed"); regenerate the diff or apply from the edits
list instead.

**Empty variable → stdin hang (2026-11-22):** `f=$(ls a b 2>/dev/null | head -1); head -2 $f` with no
match runs `head` on stdin and blocks until the 120 s timeout. Locate files with `find`/a fixed path
first, or guard with `[ -n "$f" ] ||` exit.
- zsh nomatch: a glob with no match (`rm -f t_*.c`) is an error and aborts the whole `&&` chain; use a fresh dir instead (2026-11-23).
