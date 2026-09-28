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

Also adds `flavor_eng.667`, a hidden event that boosts fertility on the current dynasty head (and
spouse) of York, Lancaster, and Tudor whenever that head has fewer than two living sons - matching
the fertility-boost pattern vanilla already uses elsewhere (`flavor_eng.46`/`.48`), rather than
spawning children directly (an earlier attempt at this that never reliably fired). It reschedules
itself every ~8-12 years, protecting every generation rather than just the current one, until the
same cutoff `flavor_eng.666` already uses (past 1500, or the War of the Roses disaster is active) -
by then the succession crisis has played out and the safety net is no longer needed.

Also adds `flavor_eng.672`, a hidden event that auto-marries any unmarried adult male (25+) in York,
Lancaster, or Tudor to a freshly created bride, and grants `+100 fertility` to both spouses.

**Why:** vanilla already has a spouse backstop for this (`flavor_eng.666`), but it only reschedules
every 400-500 months (33-41 years) and explicitly excludes heirs (`is_heir = no`). A long-lived
dynasty head can leave his own heir apparent sitting unmarried into his 30s or 40s waiting for that
rare roll - by the time it (maybe) fires, there may not be enough campaign left to produce a next
generation before the head dies, dead-ending the line despite there being a living adult son the
whole time. This isn't a court-size limit or any other hidden cap, it's simply too slow and too
narrow a net for how critical these two lines are. `.672` reschedules every ~3 years instead, covers
heirs and Tudor too, and bakes the fertility boost straight into the marriage (matching vanilla's own
`.45`/`.46` founding-event pattern) rather than depending on `flavor_eng.667`'s fertility boost, which
only ever applies to the current `dynasty_head` - not to a newly-married heir who isn't head yet.

## How vanilla engineers the Wars of the Roses

Worth understanding since it's the context the bug undermines: vanilla doesn't leave the Hundred
Years' War/Wars of the Roses arc to chance, it scripts it end to end.

- `flavor_eng.47` force-creates John of Gaunt and Edmund of Langley as sons of Edward III if he
  doesn't already have 3+ living adult sons — guaranteeing both cadet-branch founders exist.
- `flavor_eng.45`/`.46` split them into House York and House Lancaster (the buggy step this mod
  fixes). Lancaster's version also force-marries its pick and grants `+100 fertility`.
- `flavor_eng.666` (hidden, recurring every 400–500 months) auto-marries any unmarried adult male
  left in York or Lancaster to a freshly created bride, so neither line dies out for lack of
  spouses. `flavor_eng.48` separately injects a `+100 fertility` Castilian bride into York.
- `flavor_eng.49` is the deliberate trigger for civil war: once both York and Lancaster have adult
  male heads, if the reigning Plantagenet is childless, its option ("Clearly this can never cause
  a war") secretly sets `add_fertility = -100` on the ruler and spouse — sterilizing the main line
  so the succession *must* pass through a cadet branch.
- The `war_of_the_roses` disaster then starts once a Plantagenet/Lancaster ruler is infertile or
  childless (or legitimacy/stability has collapsed) with both cadet dynasties alive; on start it
  force-creates a claimant for either side that lacks a living adult male, so the war always has
  two sides, and ends when one side's dynasty head goes non-adult with no rebels left.
- Resolution into House Tudor is scripted separately: `flavor_eng.104` marries a York king's
  brother to Margaret Beaufort (Lancaster-descended via House Beaufort), producing Henry Tudor;
  `flavor_eng.105` has them raise a pretender rebellion (Bosworth) to seize the throne.

The bug this mod fixes — `.45`/`.46` only *weighting* Edward III descent instead of requiring it —
is the one weak link in that otherwise heavily-scripted chain, where bad RNG at the 1337 bookmark
can defeat all of the above before it even starts.

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
