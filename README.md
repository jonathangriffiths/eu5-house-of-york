# Fix House of York

EU5 mod making the York/Lancaster/Tudor Wars of the Roses chain survive RNG. All events live in
`in_game/events/DHE/0000_HOY_flavor_ENG.txt`.

## Vanilla

Vanilla scripts the arc end to end: `flavor_eng.47` guarantees Edward III's sons (Gaunt, Langley),
`.45`/`.46` found York/Lancaster, `.666` auto-marries unmarried York/Lancaster men, `.48` adds a
fertile Castilian bride to York, `.49` sterilizes the main line to force the civil war, and `.104`/`.105`
produce Henry Tudor and Bosworth.

## Problems

- **Wrong founder:** `.45`/`.46` only *weight* candidates toward being Edward III's son. At 1337 none
  are adult, so either can pick Thomas of Brotherton, who has no living son. York dies out at once.
- **Slow marriage net:** `.666` reruns every 33–41 years and excludes heirs and Tudor. Unmarried
  heirs can sit single into their 40s, and the line dead-ends despite a living adult son.
- **Thin lines:** nothing protects York, Lancaster or Tudor from having too few sons.
- **No Black Prince heir:** he can end up with no living son.

## Fixes

- `.45`/`.46` (overridden): candidate must descend from Edward III (`any_ancestor`), not just be
  weighted toward it. The founders are then Edmund of Langley and John of Gaunt. Known tradeoff: if Edward
  III's whole line dies out with no eligible adult, the events don't fire.
- `.672` (new, hidden): every ~3 years, marries any unmarried adult male (25+) in York, Lancaster or Tudor,
  heirs included, to a fresh bride and gives both spouses `+100 fertility`.
- `.667` (new, hidden): every ~8–12 years, gives the dynasty head and spouse a fertility boost when the
  head has fewer than 2 living sons. It stops after 1500 or once the War of the Roses disaster is
  active, the same cutoff as `.666`.
- `.671` (new): if the Black Prince has a spouse but no living son, he gets one.

## If you want the Wars of the Roses

Vanilla `.49` (1400–1500) starts the civil war by sterilizing the reigning Plantagenet ruler and spouse.
It only fires if the ruler is Plantagenet, married, has **no children** and the spouse isn't pregnant,
and both York and Lancaster have adult male heads. If the ruler has any child, `.49` never fires and
the war usually doesn't happen. The mod doesn't touch this. To get the war, don't give your Plantagenet
ruler an heir. To avoid it, have one and marry him young.

## Implementation notes

- Events can't use `INJECT:`/`REPLACE:`, so they're redefined under the original IDs.
- EU5 keeps the **first**-loaded definition of an event ID. The file is named `0000_HOY_...` so it
  sorts ahead of vanilla's `flavor_ENG.txt` and wins. It declares its own `namespace = flavor_eng`.
  The "Duplicated event ID" line in `error.log` is harmless.
- Tradeoff: the file embeds vanilla `.45`/`.46` and needs re-syncing if Paradox changes them.
