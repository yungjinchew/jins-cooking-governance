# Recipe Governance — Meal Workflow (v3.11)

> Load in addition to `recipe_governance_core.md` when generating two or more components served together.

---

## Meal Workflow

Produce a standalone Meal Workflow document in addition to individual recipes. This is the controlling execution artifact — it supersedes timing notes in individual recipes where they conflict.

**Workflow complexity budget:** Each added component, garnish, or workflow branch must solve a distinct failure mode or materially improve the eating experience proportional to its operational cost.

**Critical path:** Identify the single sequence governing final plating timing. All other components align to it. State it explicitly at the top of the Meal Workflow.

**Failure prioritization:** Identify which component degrades first if timing slips; state explicitly what to protect. Decide in advance (e.g., "pork crust is the critical path anchor — mash temperature yields if the sear runs long").

**Conflict resolution:** Use directive language — not conditional preferences (e.g., "do not delay the pork; serve immediately; accept soft potatoes" — not "soft potatoes preferred over delaying pork"). State the sacrifice explicitly.

**Oven sequencing:** Temperature transitions in order. Identify which component owns each oven state; state what yields when temperatures conflict. Where oven temperature changes between components, state: transition time required, which component holds during it and how, and when the raise begins relative to the prior component's exit. Do not assume instantaneous transitions.

**Parallel vs. sequential tasks:** State explicitly what runs concurrently and what must wait. Flag any step that cannot safely overlap.

**Attention budget:** Concurrency is limited by the cook, not only by heat sources. In the final convergence window (critical-path rest through plating), count the hands-on stations and their physical separation. Where more than two need active attention at once, or active stations sit in different locations, move a secondary component to a passive method (oven, hold) or complete it before the critical-path exit.

**Holding chain:** For each component completing before service — holding vessel, temperature, covered/uncovered, max hold time. For each drift risk, state the corrective action (e.g., "mash will tighten after 20 min — loosen with warm milk before plating" — not "mash may thicken"). **A hold ends when the component reaches the plate or the table.** Measure every hold — in the holding chain and in the variance-band check — from the component's finish to the moment it is actually plated, counted in the plating order: a component plated last waits longest, so its hold runs to the end of the plating window, not to its start. A sauce, bread or condiment that stays on the table holds through service. Never stop the count at the start of plating or at the moment the next component needs the station.

**Reserved-portion diversion (planned leftovers):** Where extra is cooked deliberately for later meals, divert the reserved portion **before** any à-la-minute finishing step rather than after. Finishes that are designed to degrade quickly — glazes, lacquers, crisped surfaces, dressed salads — should not be applied to a portion that will be stored; they store poorly and are better rebuilt fresh on reheat. State in the workflow: at which step the reserve is pulled, that it is pulled unfinished, how it is cooled and stored, and the reheat method that rebuilds the finish. Where the finishing sauce or dressing is also needed later, split it into an explicit leftover-reserve portion at the point of making it. Reserved portions require their own holding-chain row. A hot item being stored is cooled under Core's two-stage cooling check (Food-safety floor), stated as a safety requirement and not as a CCP: the CCP severity scale is culinary. Its checks are timeline rows (Timeline format).

**Sides scope for planned leftovers:** Scale only the components whose leftover value is real. Do not reflexively scale sides that degrade within a day (dressed salads, crisped or charred items) — state that they remain sized for the served meal and why.

**Final plating window:** Order of plating, time available between first plate and service, what fails first if the window is missed.

**Timeline format:** a T-minus table, one row per step (`| Time | Action |`). The Time cell is a T-minus time (T−90; a band as T−21 to T−15), a clock time for a step on an earlier day (Sat 5:00 pm), or an offset in minutes from a marked event: the anchoring component's exit (E+5; a band as E+5 to E+8; decimals allowed, E+1.5), or another event the workflow marks in its own row (P+20 after "P — purée into the Tiger"). Prose, unordered lists and non-chronological tables do not satisfy this requirement: cook mode and the print view's clock column read only the table. For any step with a defined duration, hold window, or maximum tolerance, state the actual duration inline in the Action cell. Example: `| T−20 | Steak exits oven — rest uncovered 13 min (sear at T−7). |` This allows the cook to track individual step durations independently if service time shifts — the T-minus locates the step in the sequence; the inline duration makes it self-contained.

**Holds and cooling checks are rows.** Every hold the cook must act on is a timeline row with its temperature where one applies and its limit, passive holds included ("Purée into the Tiger, Keep Warm, lid closed; above 60°C / 140°F; up to 60 min"). For food that is kept, the cooling checks are timeline rows: the start, the 1 h check, ≤21°C / 70°F by 2 h and ≤5°C / 41°F by 6 h, each with its reheat-or-discard branch (Core, Food-safety floor). The plating steps are the last rows.

**Timeline arithmetic verification:** Verify in both directions before finalising: (a) forward — each timestamp consistent with the recipe phase duration; (b) backward from service — no hold window exceeded. Flag any unverifiable timestamp. Do not assume buffer time unless explicitly named.

**Variance-band closure (mandatory where a component has a wide completion band):** Where a component's cook time is stated as a range feeding a fixed service time, the backward check must be run against **both ends of the band**, not a single assumed finish:

- **Earliest finish** → the resulting hold must be within the component's stated hold ceiling. This is the check most often missed: an early finish produces the *longest* hold, and it is the binding constraint.
- **Latest finish** → the remaining time must still cover the full convergence sequence.

If either end fails, the plan does not close. Resolve by one of: extending the hold ceiling (and labeling it designed, not validated, per Core "Designed ≠ tested"), moving the entry anchor, or narrowing the stated band. **Do not mark the check passed while the verification text itself describes an overrun** — a ✓ over a named contradiction is a governance failure, not a rounding tolerance.

**Band-edge honesty:** State the residual case once, accurately, and only if it is genuinely outside the stated band. Do not manufacture a residual gap by treating a stated ceiling as simultaneously the ceiling and something that can be exceeded — if the band is the validated bound, the plan closes at the band edge and no residual exists. If exceptional cases can exceed the band, say so in the recipe's timing line ("exceptionally thick racks may take longer; start 15–30 min earlier"), and treat it as a prep decision rather than an open timing hole.

**Strategy selection — early completion vs. just-in-time:** For any long collagen, braise, or low-roast cook whose completion cannot be timed to the minute, design the workflow around **early completion plus a safe hold**, not just-in-time finishing. State the strategy explicitly in the Critical Path. Where an extended hold is expected, pair it with a pull-point instruction — pull at the first clear threshold rather than cooking to the far edge, since the hold itself continues the texture drift.

**Warm-hold and transition interaction:** Where a component is held in the oven and the oven must subsequently change state (e.g., to broil), account for the climb from the hold temperature. Do not assert a fixed transition duration from a warm cavity; gate the next step on an observable condition (element visibly hot) and overlap the heat-up with a parallel task (surface drying, resting) so it does not block. Prefer a hold temperature that shortens the subsequent climb where the hold's primary job is keeping the item safely hot.

---

## Shared Prep

Before generating individual recipe steps for a multi-component meal, identify all ingredients that appear in two or more components. Consolidate their preparation into a single upfront Shared Prep block at the top of the meal document.

Format each line as: **Ingredient: total quantity needed (breakdown by dish)**

Example:
> Garlic: mince 8 cloves total — 3 → Stew, 3 → Marinade, 2 → Sauce
> Limes: juice 4 limes total — 2 → Stew, 1 → Salad dressing, 1 → Condiment
> Ginger: grate 30 g / 2 tbsp total — 20 g → Stew, 10 g → Sauce

Shared Prep covers only prep tasks that can sensibly be done in aggregate (mincing, juicing, grating, slicing, zesting). Do not aggregate ingredients that require different prep treatment per dish (e.g., garlic sliced for one dish and minced for another — keep those separate).

The Shared Prep block appears before the individual dish recipes in both the chat output and any Word document generated.

---

## Shopping List

For every multi-component meal, generate a consolidated Shopping List after the Meal Workflow and Shared Prep. **For single-dish requests, the ingredient list within the recipe is sufficient — do not generate a separate Shopping List.**

**Scope:** this is the Shopping List in the chat output. A printed shopping list (the app's print view, or a Word file) follows `recipe_output_print_protocol.md` §3.4 and is rendered from the saved rows, never retyped.

**Format:** A cross-tabbed table with one column per dish and a Total column. Rows are grouped by section. This makes at-a-glance cross-referencing possible when shopping and confirms that quantities have been correctly consolidated across dishes.

```
| Ingredient | Total | [Dish A] | [Dish B] | [Dish N] |
|---|---|---|---|---|
| **SECTION HEADER** | | | | |
| Item *(pantry check)* | consolidated qty | dish qty or — | dish qty or — | — |
| Potato — buy waxy (Yukon Gold, Nicola); **not Russet** | 750 g | 750 g | — | — |
```

**Buying spec on the line:** Every row whose dish depends on a variety, cut, grade, size or brand carries that spec in the Ingredient cell, and a bold **not …** where the common substitute fails (Core §III Purchase spec). The same ingredient bought to two different specs is two rows (e.g., waxy potatoes for a rösti, floury for a purée) — never consolidate rows whose specs differ. A shopper reading only the row must buy the right thing.

**Dish column naming:** Use a short identifier per dish (e.g., Stew, Salad, Cake) — not the full dish title. Where an ingredient appears in a sub-component of a dish (e.g., a garnish or sauce within a recipe), note it parenthetically in that dish's column: e.g., "2 tbsp (gremolata)".

**Total column:** States the consolidated quantity to purchase. Where quantities differ in unit type across dishes (e.g., grams in one dish, cups in another), state both: e.g., "210 g + 2½ tbsp".

**Sections** — include only those with items, in this order:
- PROTEIN / MEAT
- PRODUCE
- BREAD (when a bread or pastry component is present as a dish)
- PANTRY / DRY
- FRIDGE / DAIRY
- OILS & LIQUIDS (significant volumes: broth, wine, olive oil used in quantity)
- SEASONING (salt, pepper, finishing salts — typically all pantry checks)

Sort alphabetically within each section. Omit sections with no items. Mark likely pantry items with *(pantry check)* rather than omitting them.

**Sourcing notes ("Before you shop"):** Above the table, as `meal.shopping_notes` (data protocol 3.6), add a brief notes block for any item where: (a) a single purchase non-obviously covers multiple uses (e.g., one parsley bunch covering salad, braise, and garnish), (b) a substitution is relevant, or (c) a quantity requires sourcing clarification. Omit the notes block if nothing material to flag. Sourcing notes explain; they never carry the only statement of what to buy or what not to buy — that belongs on the row.

Include rice in the shopping list once type and quantity are confirmed, normally marked (pantry check). If type is unresolved, list “rice — type to confirm” and do not finalise the meal until it is resolved.

Quantity reconciliation: Before finalizing, verify that every Shared Prep total and Shopping List total equals the sum of its dish allocations after unit conversion. Where a purchase unit differs from recipe-use units, state both the purchase quantity and the allocated recipe quantity. Any unresolved mismatch, omitted allocation, or double-counted reserve is a workflow failure.

---

## Cooking Sheet Format (Multi-Component Meals)

The printed sheet follows `recipe_output_print_protocol.md`: §3.1 lists what of the Meal Workflow prints and what stays in the database, §4 the page plan. Two authoring rules follow from it: the Critical Path names what yields first if the timeline slips (the sheet prints no failure-prioritisation section), and the plating steps are the last rows of the T-minus timeline (the sheet prints no plating-window section).

---

For single-step or single-ingredient preparations, these sections may be declared Not applicable without justification: Prep Phase (if part of Active Cooking), CCPs, Adjustment Notes, Scaling Notes. Flavor Architecture and Holding & Assembly remain required. Active time must still be stated.

**Condiment role:** A condiment must either externalize variability (heat, acid, salt) or provide textural contrast. Do not add one that duplicates the dominant flavor profile of another component.

---

## Meal-Level Awareness

- Avoid redundancy in technique or richness across dishes
- Flag any dish occupying the oven or a burner for extended time that affects parallel cooking feasibility
- Balance across richness / freshness / texture axes

**Meal balance audit:** State how the meal manages richness, acidity, heat, temperature, and texture. Identify any accumulation risk — fat saturation, sweetness stacking, acid fatigue, spice build, smoke/umami overload — and name the relieving component. If no relief exists, add one or flag the gap. Where the same primary acid source appears in three or more components, confirm each performs a functionally distinct role (depth vs. brightness vs. reset vs. cleanse); duplicate function without distinct purpose is meal-level redundancy — resolve or explicitly accept.

**Meal-level texture exemption:** Inherently single-texture components (custards, mousse, panna cotta, purées, soft cakes, pure soups) are exempt from the two-texture requirement when the broader meal provides sufficient contrast progression. Invoke with: "Meal-level texture exemption: [component] is inherently single-texture; contrast provided by [other component(s)]."

**Dessert function:** Evaluate against accumulated richness, palate fatigue, residual spice, and alcohol pairing. State whether dessert functions as continuation, contrast, or reset — and whether the accumulated meal load supports it.

**Emotional register:** Where the user states an intended register — comforting, celebratory, restrained, luxurious, rustic — preserve it while balancing richness, freshness, and texture.
