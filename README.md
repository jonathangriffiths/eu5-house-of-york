# Fix House of York Founding

An EU5 mod fixing a vanilla bug in the dynamic historical events that found House of York
(`flavor_eng.45`) and House Lancaster (`flavor_eng.46`), in `in_game/events/DHE/flavor_ENG.txt`.

## The bug

Both events only *weight* their Plantagenet-side candidate toward being a son of Edward III
(`factor = 100` if `father ?= character:eng_edward_iii`) rather than requiring it. At the 1337
bookmark, none of Edward III's own sons are adults yet, so both events can end up picking
`eng_thomas_of_brotherton` — the only other eligible male Plantagenet alive, and a dead end with no
living son. For York this is fatal: the new dynasty dies out almost immediately. Lancaster survives
because its version force-marries the pick and grants `+100 fertility`, but it's still the wrong
historical founder.

## The fix

Changed the weighting into a hard requirement: the candidate must be a descendant of Edward III
(using `any_ancestor`, not just `father ?=`), applied to both events' `trigger` and
`random_character_in_dynasty` selection. This matches the real historical founders (Edmund of
Langley for York, John of Gaunt for Lancaster) and keeps working for later generations as a campaign
runs on. Everything else about both events is untouched vanilla behaviour.

**Known tradeoff:** if a campaign diverges enough that Edward III's entire line dies out before
producing an eligible adult, these events simply won't fire — there's no fallback to the wider
dynasty (vanilla would still attempt something, buggy as that is).

## Why these files redefine the events under their bare ID, not `INJECT:`/`REPLACE:`

Events aren't part of the merge-aware "database" system that `INJECT:`/`REPLACE:` belong to (unlike
laws, gods, country modifiers — see the sibling `eu5-fix-trade-loops` mod). The correct way to
override an event is to redefine it under its original ID (`flavor_eng.45 = { ... }`,
`flavor_eng.46 = { ... }`).

That alone isn't enough, though: for a given event ID, EU5 keeps the *first*-loaded definition and
silently discards later ones (logged as a harmless "Duplicated event ID" error.log entry) — the
opposite of the usual "last write wins" assumption. All game files, vanilla and mods alike, are
merged per-directory and read in ascii filename order, so this mod's override file is named
`0000_HOY_flavor_ENG.txt` to sort ahead of vanilla's `flavor_ENG.txt` in `in_game/events/DHE/` and
therefore load — and win — first. Because it now loads before vanilla's file, it also declares its
own `namespace = flavor_eng` rather than relying on vanilla having already declared it.

Tradeoff: this embeds vanilla content and needs re-syncing if Paradox changes these events.
