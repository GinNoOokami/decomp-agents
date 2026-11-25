---
name: flag-sets-as-enums
description: "bit-flag sets go in a typedef enum (Name = 0x1, ...), not bare #defines; derived masks/ranges can be #defines"
metadata:
  type: feedback
---

_Carried over from ff7-decomp on 2026-10-30; examples are from FF7, the rule applies to FFX._

When introducing a set of named bit flags (e.g. item restriction bits), write a `typedef enum { ITEM_RESTRICT_SELL = 0x1, ... } ItemRestrictionFlags;`, not a list of `#define`s.

**Why:** 2026-10-23 I added `ITEM_RESTRICT_*` as four `#define`s; the user: "These should've been a enum list, not bare defines".

**How to apply:** flag sets and other value sets → typedef enum in the domain header. Derived combinations (`BATTLE_ITEM_RESTRICT_MASK`) and id ranges following game.h's `X_START`/`X_END` + `NUM_X` pattern stay `#define`s. Put defines with their domain header (item ids/flags → `include/item.h`; battle-only masks → `battle.h` when batini/batres also use them, else `battle_private.h`). Related: [[naming-preferences]], [[project-style-conventions]].
