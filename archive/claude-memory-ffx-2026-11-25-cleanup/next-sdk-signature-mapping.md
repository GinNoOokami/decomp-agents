---
name: next-sdk-signature-mapping
description: "user's plan after read_debug.c — identify the SDK library functions (sceOpen etc.) with our own evidence before more game code"
metadata:
  node_type: memory
  type: project
  originSessionId: d8b8ba71-d5b2-44ee-a8a4-83ea356c0bf1
  modified: 2026-10-30T23:26:20.277Z
---

After `read_debug.c` is finished, the user wants to map the remaining SDK library functions (e.g. 0x2FC1A0–0x2FCD10, so far named only from Rozelette's symbols: sceOpen/sceLseek/sceRead/sceClose/sceIoctl) so later matching works from confirmed names and signatures. Stated 2026-10-30.

**Why:** names from Rozelette's `symbol_addrs.txt` are leads only ([[community-tools-corroborate-only]]); I used one (sceOpen) as fact and the user asked where it came from.

**How to apply:** when read_debug.c is done, raise this as the next step. Candidate methods: library fingerprinting against an SDK build ([[decomp-reference-techniques]]), call shape (SIF RPC for file I/O), PsIIlib ID strings. Until then, say "Rozelette's name, unverified" whenever I use one of those names.

**Progress (2026-10-30):** user's rule: our own minimal SDK headers in `~/dev/ffx/include/` named after Sony's (`sifdev.h` first), SDK 2.4.2 headers (GirianSeed/globe repo, branches per release, Sony-confidential: reference only, never copied) as reference, only functions FFX is confirmed to call; append as we confirm more. sifrpc.h deferred until a game caller of sceSifCallRpc/BindRpc is decompiled (needs sceSifClientData layout check). Method that worked: SIF RPC server ID + function number per wrapper. Next: libcdvd the same way.

**Progress (2026-10-30, later):** done on main: sifdev.h, libcdvd.h, libmc.h, libpad.h, eetypes.h, libvu0.h + symbol_addrs groups. Sony library objects match when built with `~/opt/ee-toolchains/ee-gcc2.9-991111-01-dtls13010` (not the game's 991111-01) and the globe Makefile flags; compare with `~/dev/ffx-analysis/scripts/objcmp.py <obj> auto|<base>`. libvu0 = 2.0 source exact (Toolchain.md). 2.5.1 libdev devvu1/devvif0 exact, devvu0 12/13, libdma 6/19 (layout differs) — not yet named. read_debug_c branch still needs its read_debug.c patch (scratch proposal/read_debug.diff, held by user).

**Paused (2026-10-30, user: diminishing returns):** SDK naming pass done for what the game calls; leftovers listed in DecompNotes FFX/SystemSplit.md "SDK libraries: what's left". IOP-side decoding method: extract modules from the disc's IOPRP234.IMG (ROMDIR) and read each RPC server's dispatch table. Resume only if the user asks or finds a symbol-bearing 2.3.x ELF.
