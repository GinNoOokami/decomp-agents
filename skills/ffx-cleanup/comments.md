# Comments

## What a comment may say

- **Only what the code can't show:** meaning of data, why a check exists, a formula's intent, what
  a bit layout holds (`// high nibble = current weight, low nibble = base weight`).
- **String globals get their contents side by side, always:** a `D_` extern holding a string
  (`.data` or `.rodata`) reads `extern char D_00316F50[]; // "btlenv.txt"`. Read the ELF bytes
  for every data global the function touches before typing it; this comment is standard, never
  an optional item. It is a stand-in: once the section is split and the literal moves inline
  into the matched C (or the C defines the string), the extern and its comment go.
- **Not what the code visibly does,** latent bugs included ("Reuses the table index" was rejected).
  Exception: a `// bug:` comment when the consequence isn't visible (TkCloseControler's negative
  port reads `D_0078A7C0[-1]`).
- **Not matching requirements.** Once merged, nobody can change a matching shape without breaking
  the build. Mention match-required locals/shapes in chat instead.
- **Not stronger than the evidence.** "battle number?" on `BtlListEntry.id` rested on one Switch
  comparison that only showed a lookup key; the code supported "low half of a battleId". Before
  writing a meaning, ask what the code (ours or Switch's) actually shows.
- **Check edge values before writing a rule.** "higher = rarer battles" was right for nonzero rates
  but 0 skips the roll: `// 0 = no random battles, otherwise higher = rarer`.
- **Self-contained:** no "it"/"when it did" leaning on another comment, no pointers to DecompNotes,
  step numbers or private files. The repo may go public.
- **Explain game context when it helps the reader** (what the mechanism does in play) only if short;
  the longer story belongs in chat or notes.

## Lead comments

- **Shape:** one short line, the hypothesis, ending in `?` (`// the controller type ID libpad
  reports?`). Evidence goes in chat, not the comment; don't restate who reads or writes the field.
- **Placement:** functions: own line above the definition (`// MsSetEventBattle?`). Fields, globals,
  prototypes: side by side (clang-format aligns them).
- **No `Lead:` prefix.** Prototypes of unmatched functions with a Switch candidate:
  `void func_001CE498(); // Switch MsEventEffectInit?`.
- **Renames drop redundant comments:** when the new name says it, delete the comment; keep only the
  extra information (`repeatDelay; // first-repeat delay?` keeps "first"). When a function's own
  code now shows what a lead claimed, cut the lead to what's left (`// battle number? Switch compares
  it…` → shorter once our C shows the compare).

## Evidence strength

- Code comments state the hypothesis with `?`; certainty words ("Certain.") never go in comments.
- A Switch name in a comment says "Switch" (`// Switch Sg_FadeInW?`); our own candidates don't.
- Leads and facts about specific code live in the code, not in notes. Notes take only cross-cutting
  findings and open investigations; a lead backed by new evidence can be promoted there once.

## The user's comments

- The rules above are for comments **I** write. The user's comments and style choices (decimal
  literals, hedged suppositions, dead-code notes) stay. Suggest improvements; change directly only
  what is false or misleading, and say what changed.
- The user's short `// getFoo?` notes from transcribing m2c are guesses, never evidence.

## Form

- `//` line comments in our code (src/, include/ outside include/sdk/), never `/* */`; struct offset
  tags `/* 0xNN */` are the exception. SDK headers keep block style.
