# Naming

## Functions

- **Switch names win.** A matched function whose Switch counterpart is checked by body *and*
  position gets the original name via `rename.py OLD NEW --switch` (symbol_addrs line with the
  standard marker, C rename, make). Only matched functions: `rename.py` refuses INCLUDE_ASM ones;
  unmatched names stay lead comments in callers' headers or the handoff.
- **Position alone is a lead, not a name.** Init/Get/Set runs (MsInitBattleEncount,
  MsGetBattleEncount, MsSetBattleEncount at 0x1AE908/950/988) were named only after each Switch body
  showed the same inlined helpers.
- **No swpair/swfill row is not "no name".** Read `swrun.py FUNC`, the full `functions.tsv` around
  named neighbours, and the Switch bodies of callers with lead names (func_001AF238 = MsGetFileSize
  was found through its caller's Switch body). Alpha strings at the boundary help place a run.
- **Inlining trap.** A much larger Switch function, a different signature (argument count, all
  ports vs one) or a string pair on a worker means PS2 code was inlined into it. The name goes to the
  PS2 *entry* with the whole role, never to an inner helper (TkMcSync, TkResetControlerWork,
  DebugPrintf cases).
- **Inlined helpers get lead comments.** A PS2 helper that Switch inlines everywhere has no symbol:
  `// Name?` above the definition (`// MsInitEncountWeights?` for func_001AD8D0). Build the
  candidate from the system's own vocabulary (`Ms…Encount`, `MsBtlRead*`); no candidate is not a
  reason to skip.
- **The verb must fit the body.** Get = returns a value; Set = stores a decided value; Init =
  restores defaults (MsInitBattleEncount only resets weights: the original's word, kept); Update =
  per-frame step. For our own names, name what changes (`InitEncountWeights` over
  `InitFieldEncount`); a flag mode the name doesn't cover is the parameter's job (`keepWeights`).
- **An original name that seems wrong is still the name.** Explain what it "inits" in chat; don't
  rename originals to fit our reading.

## Parameters and locals

- **Pass-through values take the callee's parameter name** (`id` into `MsBtlListFieldNum(id)`,
  `mode`/`timeout` into an SDK call). This is trivial and applies directly.
- **Packed values get a name for the whole and the parts:** `battleId` = `fieldId << 16 | entryId`
  (func_001AD9C0). Don't reuse a word that already means something else in the function
  (`encounter` was the table, so not `encounterId`).
- **Out-parameters:** plain names by default (MsBtlListCDpos `s32* groupIndex`). Only when a
  descriptive local holds the same value, the parameter gets an `out` prefix (`outGroupIndex`).
  Never `-Out` suffixes or `p` prefixes.
- **Plural for an array base that gets indexed** (`entries`, `&entries[n]`), **singular for a
  pointer that walks** (`entry++`). Apply across the TU.
- **A variable reused for a second meaning** (count, then chance out of 256) keeps one name when the
  match needs one variable; a short comment on the reassigning line explains the switch.
- **Full words** unless the original names abbreviate (`Btl`, `Ms`, `Num`, `CDpos` are original):
  `fieldOffset`, not `fieldOfs`; `Debug…`, not `Dbg…`.

## Fields and globals

- **Rename every use in the same change.** Grep `\bOLD\b` over `src/` and `include/` before and after
  (the `unk1986` rename missed `MsBattleGetNum` and broke the build). `rename.py --data` for globals.
- **Prefix by area in big structs.** In the 0x2148-byte battle state, encounter fields read
  `encountType`, `encountRate`, `encountEnabled`, `encountRollDist`, `encountDist`: a bare
  `rollDist` or `lastRate` says nothing next to 200 other fields.
- **Fields a function stores together get one vocabulary** (`fieldIndex`/`groupIndex`/`entryIndex`
  for the battle to run, matching the BtlList functions), with one block comment if they form a
  group.
- **A getter with an original name names its field** (`MsGetEncountType` → `encountType`).
- **One vocabulary per system:** when names move to a new vocabulary, all identifiers of the system
  move at once (functions, globals, fields, defines, header entries, symbol_addrs). Ambiguity a new
  name creates is resolved in the same change.
- **Our earlier names are best guesses, not evidence.** Before citing a name in the tree as support,
  check who introduced it (`git log -S`); revisit it when new facts arrive.
- **Vague beats hex** (FFX policy): name from what the code shows and mark the guess with `?`;
  keep the address name only when nothing is known.
- **Settled globals keep the `D_` name** with a side-by-side lead comment (`// gTkName?`, or the
  Switch name: `// Switch save_ram? (+0x3D14 = battle count)`).
