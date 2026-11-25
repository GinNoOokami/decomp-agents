# Order and scheduling differences

## First: the order you see is not the source order

- m2c's statement order and the target's store order are scheduler output. Before any RTL or
  aliasing theory, try natural shapes: related assignments grouped, copies in field order, flag
  setters together, siblings' and refs' shapes (func_0011F8F8: the matching source put
  `D_00307C60 = 1` first, fifth in the asm, and the five stamp copies together in field order; an
  "it's aliasing" theory had been built on m2c's order).
- A statement stuck where the asm shows it: try its natural positions (Sg_ScreenshotWrite: `written
  += 0x600` sits in the row loop's header in the asm, at the row's end in the source).
- **Brute force with `vsweep.py orders`/`moves`** before reasoning about allocation (func_00163450: 120 orders,
  one matched (only lsn, size, buff first); func_00165588: 110 single moves of an 11-store block, one
  matched). Cover every statement and report coverage ("N of M orders"); a sweep with the first line
  fixed proves nothing about order (func_0011F8F8: `--from` on line 2 and 4000 of 9! sampled, then
  "not order" was claimed; the fix moved another statement first).
- When several orders match, prefer the PC port's order (func_00163D20, func_00164A70 end stores).
- **Store hint:** the value loaded last goes in v1 and is stored first.

## Reading order constraints from the asm

- **Global stores: `lui`s come out in reverse source order**; source in call order matched
  (func_002556F8).
- **Under -fno-strict-aliasing a global-table load can't move above a store through a struct
  pointer**, so a table load before a field store in the target was before it in the source. Use it
  to prune order searches. (Not in conflict with the next rule: an indexed table is a varying
  address, not a fixed scalar; inferred.)
- **Struct-field reads through a pointer get hoisted above fixed global stores** (alias.c
  `fixed_scalar_and_varying_struct_p`): `p->f` and `p[1]` count as struct, plain `*p` doesn't. Loads
  staying below such stores = plain derefs (movie_start_debug_resulti; its cursor-copy shape matched
  but is unconfirmed as source style).
- **A store through a call result into a struct field** (`((TaskSlot*)create(...))->args[0] = G;`)
  lets gcc keep global G in a register afterwards; a plain `*(T**)p = G` forces a reload
  (func_00159CC8; struct-member stores treated as not aliasing plain globals, inferred).
- **Byte stores force a global-pointer reload:** through `G->field` gcc reloads G after every `sb`, not
  after `sw`. Group the target's stores by which reload they use: each group is one source run, word
  stores first, its byte store last (func_0015F7E0: 143 -> 44 differing in one step).
- **A flag polled in a wait loop** needs `extern volatile` (func_00255158).

## Alias sets and options

- **Field types steer sched1 through alias sets** (func_00162DE0): RTL `(mem/s:SI ... N)` carries the
  set; stores of different types don't conflict, which drops a dependence and changes priorities.
  c-common.c `c_get_alias_set`: signed/unsigned ints share a set, each pointer type has its own,
  union-member and bitfield accesses get set 0. A union field matched; whether the original had it
  is open. When natural orders and sweeps have failed, read the `-dS` ready lists and alias numbers
  before more order guesses.
- **An order difference that survives every natural shape** may be option-controlled: that is how
  `-fno-strict-aliasing` was found (T3; func_00164320 stuck at 190/194 on a callback reload vs
  `read_q_req_no` load; it also explained the "tentative union" in `ReadQEntry.buff`). A flag is a
  whole-build hypothesis: diff every C file with and without it; the user edits the Makefile. Flags
  can differ per library (libmscd vs game code), so retest new files both ways. variant.py hard-codes
  CFLAGS; keep it in sync.

## Scheduler internals

- **sched1 rank order** (haifa-sched.c `rank_for_schedule`): priority, then INSN_REG_WEIGHT (sets
  minus REG_DEAD/UNUSED, lower first, sched1 only), then relation to the last scheduled insn, then
  dependent count, then luid.
- **PRE-inserted loop decrements** (func_0011F2B8, parked): gcse turns a latch's `i--` into
  `reg = i - 1` at the end of the header block (`daddu v1,t0` pattern); it kills `i` (weight 0) and
  jumps ahead of a same-priority `x += c`. Source order inside the header can't change this; read
  the `.gcse` PRE lines and the weights first.
- **sched1 won't hoist an insn above a call** only if its dest crosses no call
  (`REG_N_CALLS_CROSSED == 0`).
- **Which loop constant gets hoisted = statement order** (func_002D1EA0, loop.c `move_movables`):
  movables go in insn order while `threshold*savings*lifetime >= insn_count`, threshold -3 per move.
  The last one moved has the shortest range and wins the last saved register; the rest are
  rematerialised in the loop. Target keeping an earlier constant (0x10000) in s8 and a later one
  (0x1000) in the loop = something took the last slot first (storing `entry`, a `%hi/%lo` address,
  before `stackSize`). Read the "moved to / not desirable" lines in `.loop`, then permute that one
  group (720 orders, ~3 min). Same function: repeated `arr[i].f` with a 0x30 struct kept the `*48`
  constant from moving, so a `ThreadParam*` local was required (81 vs 72 insns in every matching
  order).
