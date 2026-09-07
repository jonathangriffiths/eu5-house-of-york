# Fix House of York Founding

An EU5 mod fixing a vanilla bug in the dynamic historical event that founds House of York.

## The bug

`flavor_eng.45` ("Founding of House of York", `in_game/events/DHE/flavor_ENG.txt`) picks a random
adult, non-ruler, non-heir male Plantagenet to found House of York. It only *weights* the pick
toward sons of Edward III (`factor = 100` if `father ?= character:eng_edward_iii`) rather than
requiring it.

At the 1337 bookmark start, none of Edward III's own sons are adults yet, and every other eligible
male Plantagenet is already dead:

- Edmund of Woodstock — d. 1330
- Edmund of Kent — d. 1331
- Edward of Norfolk — d. 1334
- John of Eltham — d. 1336

That leaves **`eng_thomas_of_brotherton`** as the only character in the entire dynasty who can match
the trigger. He has no living son (his one son, Edward of Norfolk, died in 1334) — only two
daughters — and nothing in the game reliably arranges his remarriage: `marry_noble.txt`'s AI urgency
weighting for arranging a new marriage only applies to rulers and heirs, neither of which he is. So
the event is essentially guaranteed to hand House of York to the wrong historical person, and the
freshly-founded dynasty dies with him almost immediately.

## The fix

Requires the candidate to actually be a son of Edward III — matching the real historical founder,
Edmund of Langley — in both the firing `trigger` and the `random_character_in_dynasty` selection in
`immediate`, instead of merely weighting toward it. This means the event can't fire until Edward III
has an adult son, which is exactly when it should (historically ~1360), rather than firing early on
whichever unrelated great-uncle happens to be available and childless.

This was chosen over the lighter alternative (just requiring the candidate to already have a living
son, whoever they are) because it fixes the actual root cause — the wrong person being picked — and
Edward III's real sons already come with their own sons in the game's character data, so the new
dynasty gets a working succession path for free rather than one bolted on by a generic filter.

**Known tradeoff:** because the trigger now hard-requires a son of Edward III, if a campaign
diverges hard enough from history that Edward III's line goes extinct before producing an adult son,
this event will simply never fire again — there's no fallback path to the wider dynasty. Vanilla's
looser trigger would still attempt something (with the same "immediately dies out" bug) in that
case; this mod trades that broken attempt for no attempt at all. If that turns out to matter in
practice, the fix could be extended with a fallback branch that reintroduces the wider-dynasty pool
gated by a "has a living son" filter.

## Why this file uses `REPLACE:` instead of `INJECT:`

The fix edits conditions nested inside the event's existing `trigger` and `immediate` blocks — an
`any_character_in_dynasty` existence check and a `random_character_in_dynasty` selection limit —
not fields that can be additively merged. Adding a second top-level `trigger = {}` or
`immediate = {}` block via `INJECT:` would be ANDed/executed independently rather than tightening
the same nested character filter (an existence check in a second trigger block only proves *some*
character satisfies the second filter, not that the *same* character satisfies both). So this mod
`REPLACE:`s the whole event with a full copy of its vanilla definition, with the fix merged directly
into the nested trigger/limit blocks. Tradeoff: this embeds vanilla content and needs re-syncing if
Paradox changes this event.

