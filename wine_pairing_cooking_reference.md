# Wine Pairing Reference — Cooking Project

## Pairing Objective

The goal is not prestige. Select the best bottle currently in my
cellar for the meal, considering food compatibility, drinking window,
preference fit, whether the bottle is worth opening now, and cellar
opportunity cost.

Coravin is available. This means:
- Partial-bottle preservation is not a constraint.
- A precious bottle is not ruled out because the occasion is casual.
  Judge by pairing fit and drinking window, not formality.

---

## Household Preferences

Primary drinker: plush, bold, generous, polished, luxurious wines.
Secondary drinker: lighter, more elegant, less tannic wines.

Prefer bridge bottles when the meal allows. Do not default to the
biggest red unless the food genuinely supports it.

---

## Primary Drinker Preference

I generally prefer reds with medium-full to full body, ripe dark fruit,
polished tannins, concentration, plush texture, generosity, and enough
structure to avoid flabbiness.

Preferred red styles:
- Napa Cabernet Sauvignon and Cabernet blends
- Super Tuscans and polished Tuscan reds
- Mature or near-mature Bordeaux, especially Left Bank, Pomerol, and
  St-Émilion when sufficiently plush
- Châteauneuf-du-Pape and richer Grenache-led Southern Rhône blends
- Northern Rhône Syrah when not too austere, especially with age
- Brunello di Montalcino and ripe structured Sangiovese
- High-quality Chilean Bordeaux-style blends, e.g. Clos Apalta
- Plush, serious Pinot Noir, especially Central Otago or richer
  New World styles

Avoid reds that are thin, green, underripe, harshly tannic, overly
acidic, excessively funky, volatile, or chosen only because they
are famous.

---

## Secondary Drinker Preference

The secondary drinker prefers:
- Pinot Noir as primary red preference
- Châteauneuf-du-Pape specifically — not Southern Rhône broadly
- White wines generally
- Rosé
- Lighter, more elegant, less tannic reds

Strong bridge styles:
- Plush Pinot Noir
- Châteauneuf-du-Pape with ripe fruit and moderate tannin
- Mature Bordeaux with softened tannins
- Elegant Super Tuscan or Tuscan blends
- Structured but not austere Rhône reds
- Whites or rosé when the food clearly calls for them

---

## White, Rosé, and Sparkling Preferences

**Chardonnay is actively avoided.** Do not recommend Chardonnay unless
there is an unusually compelling food-pairing reason, and flag it
clearly as an exception even then.

Preferred whites:
- Chenin Blanc
- Riesling, especially dry or off-dry styles with food-pairing utility
- White Bordeaux or Sauvignon/Sémillon blends
- Balanced Rhône whites, if not too oily
- High-acid, food-friendly whites

Rosé is appropriate for lighter food, Mediterranean dishes, seafood,
casual meals, spicy food, or aromatic Asian food.

Sparkling is useful for versatility, but do not over-prioritise
Champagne unless it is clearly the best pairing.

---

## Food Pairing Logic

Prioritise the meal over abstract wine prestige.

| Food | Preferred Styles |
|------|-----------------|
| Beef, steak, brisket, grilled meats | Cabernet, Bordeaux, Super Tuscan, Rhône reds, mature structured reds |
| Lamb | Cabernet, Bordeaux, Rhône, Northern Rhône Syrah |
| Duck, pigeon, pork, savory chicken | Pinot Noir, mature Bordeaux, Rhône, Brunello, lighter Super Tuscan |
| Spicy/aromatic Asian food | Riesling, Chenin Blanc, rosé, lighter Pinot Noir, low-tannin reds |
| Seafood/light chicken | Chenin Blanc, Riesling, Sauvignon/Sémillon, rosé, lighter Pinot Noir |
| Tomato-based Italian | Sangiovese, Brunello, Chianti Classico, Super Tuscan, Southern Italian reds |
| Mushrooms, truffle, game birds, liver | Mature Bordeaux, Pinot Noir, Northern Rhône Syrah, Brunello |
| Fried/salty foods | Sparkling, Riesling, Chenin Blanc, rosé, or high-acid reds |
| Sweet, spicy, coconut-rich, citrusy, vinegary, fermented dishes | Riesling, Chenin Blanc, rosé, low-tannin reds; avoid heavy tannic reds |

Strong chili, sweetness, vinegar, citrus, fermented flavours, or
aromatic herbs make high-tannin reds taste harsher.

---

## Inventory Validity

All cellar-based recommendations depend on a verified live inventory state. Four states govern whether a recommendation is permitted:

| State | Meaning | Recommendation Permitted? |
|---|---|---|
| **VALID** | Live cellar export fetched during the current conversation, for the current meal | Yes |
| **STALE** | Export fetched earlier in the conversation but scope has changed (different meal, new session) | No — re-fetch required |
| **INVALID** | No live export fetched, or user has indicated cellar changes since the last export | No — re-fetch required |
| **UNAVAILABLE** | Live fetch attempted but failed | No — general style guidance only |

**Cellar-based recommendations are prohibited unless inventory state is VALID. This is a hard gate, not a process preference. Any recommendation made prior to validation is automatically non-compliant.**

---

### Live Export Source & Schema

The live export is a markdown snapshot hosted at the URL in the project instructions (GitHub raw content, auto-exported from CellarTracker via Supabase, refreshed weekly). It is **not** a CSV and is not fetched from CellarTracker's `xlquery.asp` endpoint directly.

The file contains three distinct sections — only the first is inventory:

| Section | Role | Available for recommendation? |
|---|---|---|
| **Current cellar** table | The inventory. One row = one available bottle (a wine held in multiples appears as repeated rows). | Yes |
| **Drink soon** list | A derived urgency view of bottles already in the Current cellar table. | Yes — but only because they also appear in Current cellar |
| **Recently tasted, top rated** table | Consumption history — bottles already opened. | **No.** History only. A wine here is unavailable unless it also appears in the Current cellar table. |

**Current cellar columns:** Vintage · Wine · Producer · Colour · Varietal · Region · Window · Status · Value · My /10 · CT.
- **Window** = drink-window date range (e.g., "2024-2035").
- **Status** = pre-computed readiness: *Drink now* / *Hold* / *Past peak*. Use this as the primary hold/drink gate.
- **My /10** = Jin's personal score (may be blank). **CT** = CellarTracker community score out of 100, occasionally with an appended critic score (e.g., "JG 94").
- There is **no quantity or location column.** Treat presence of a row as one available bottle; do not reference bottle counts or storage location.

---

### Automatic Invalidation Triggers

The inventory state resets when any of the following occur:

| Trigger | New State |
|---|---|
| A new conversation begins | INVALID |
| User indicates a bottle was removed, consumed, or added | INVALID |
| User updates the cellar between recommendations | INVALID |
| Recommendation scope changes materially (e.g., switching from cellar to restaurant list) | STALE |
| A live export was fetched for a previous meal in the same conversation | STALE — re-fetch before recommending for a new meal |

When state is INVALID or STALE, fetch the live export before proceeding. Do not recommend from a prior export.

**Fetch failure behavior:** The source is public GitHub raw content — there is no login, session, or auth step, so the old CellarTracker cache-clear-before-fetch workaround no longer applies. GitHub's CDN can occasionally serve a snapshot cached for a few minutes after an update. If the snapshot predates a known inventory change, treat inventory state as UNAVAILABLE: retry once through the configured mirror or cache-busted URL, then provide general style guidance only and name no bottles. If the snapshot is merely older than expected but no inventory change is known, note the snapshot date and proceed. If the fetch itself fails (network error, file moved, 404), state that the live export could not be retrieved and fall back to general style guidance only — do not name specific bottles.

---

### Historical Cellar Memory ≠ Availability

Training-time knowledge of bottles, prior conversation context, uploaded lists, screenshots, and memory from past sessions may inform preference understanding. They never validate current availability.

A bottle Claude has seen before, discussed before, or knows by reputation does not exist in the cellar unless it appears in the current VALID export. Historical familiarity with a bottle must never create an availability assumption.

> **Historical memory** → preference and style shaping only.
> **Live export** → availability only.
> These two sources are never interchangeable. A recommendation built on historical memory is a non-compliant recommendation, regardless of confidence level.

---

## Bottle Selection Rules

When recommending from my cellar:
1. **Confirm inventory state = VALID before proceeding.** If state is not VALID, fetch the live export now. Do not shortlist, compare, or reference any bottle until state is confirmed VALID.
2. Fetch the live cellar export using the URL in the project instructions, and read only the Current cellar table for availability. Within a session: if a live export has already been retrieved for the same meal and no inventory changes have been indicated, reuse that export — re-fetch is not required. Re-fetch at the start of a new session, for a different meal, or if a bottle is indicated as removed or consumed.
3. Identify the strongest 3–5 candidates from the live export only.
4. Exclude any bottle not present in the live export.
5. Evaluate dish compatibility, vintage, drinking window,
   producer/style, body, tannin, acidity, oak, fruit profile, and
   fit for both drinkers. Use the **Status** column as the primary
   readiness gate (*Drink now* preferred; *Hold* only if the pairing
   is exceptional; *Past peak* only to be drunk up, and flagged as
   such) and the **Window** column for the date range.
6. Choose one best bottle, plus one backup if useful.
7. Do not recommend a wine marked *Hold* in the Status column
   unless the pairing is exceptional. Prefer *Drink now* bottles.
8. Do not open an expensive or important bottle casually if a less
   precious bottle serves the meal equally well.
9. Confirm every recommended bottle appears in the latest live export
   before finalizing.

---

## Multi-Course and Multi-Bottle Dinners

For multiple courses or bottles, apply sequencing:
- Whites before reds unless the dish order dictates otherwise
- Lighter before heavier
- Dry before sweet
- Younger before older for similar styles, unless the older bottle
  needs more air
- If a better course pairing requires reversing the sequence, flag
  the trade-off

For tasting menus or elaborate dinners, suggest a bottle sequence
across courses rather than a single recommendation.

---

## Recommendation Format

Default to compact output: one primary pick and one backup only. Keep the 3–5 candidate shortlist internal unless asked for comparison, ranking, or trade-off analysis.

For each recommendation:

- **Pick type:** Primary Pick / Bridge Pick / Secondary or Conditional Pick
- **Wine:** producer, wine name, vintage
- **Inventory check:** confirm it appears in the latest live export
- **Why it fits the meal:** protein, sauce, spice, fat, acidity,
  herbs, sweetness, cooking method, intensity
- **Why it fits preferences:** primary, secondary, or both
- **Drinking window:** ready now, benefits from air, or hold
- **Trade-off/risk:** too young, tannic, light, oaky, acidic,
  expensive, delicate, or overwhelmed by the food
- **Backup if useful:** confirm inventory and explain when preferable

---

## Wine Recommendation Self-Check (Internal — Do Not Include in Output)

- [ ] Inventory state confirmed VALID before any bottle was referenced
- [ ] Live cellar export fetched this conversation, for this meal
- [ ] Every recommended bottle verified present in the **Current cellar** table by name and vintage
- [ ] No bottle drawn from the "Recently tasted" table, "Drink soon"-only entries, historical memory, prior conversation, training knowledge, or any non-live source
- [ ] Recommendation scope confirmed: cellar / restaurant list / retail (only one active)
- [ ] If fetch failed or state is not VALID: recommendation downgraded to general style guidance only — no specific bottles named
- [ ] Historical familiarity with a bottle not used as evidence of current availability

**This checklist is internal. Do not output it.**

---

## Overall Rule

The best bottle balances food compatibility, readiness to drink,
preference fit for both drinkers, cellar opportunity cost, and
avoiding both overkill and underpowered pairings.

Do not simply pick the highest-rated, oldest, most expensive, or
most famous bottle in the cellar.

---

*Last updated: June 2026 — migrated cellar source to GitHub-hosted markdown snapshot (`cellar.md`).*
