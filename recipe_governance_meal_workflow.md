# Recipe Governance — Meal Workflow (v3.4)

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

**Holding chain:** For each component completing before service — holding vessel, temperature, covered/uncovered, max hold time. For each drift risk, state the corrective action (e.g., "mash will tighten after 20 min — loosen with warm milk before plating" — not "mash may thicken").

**Reserved-portion diversion (planned leftovers):** Where extra is cooked deliberately for later meals, divert the reserved portion **before** any à-la-minute finishing step rather than after. Finishes that are designed to degrade quickly — glazes, lacquers, crisped surfaces, dressed salads — should not be applied to a portion that will be stored; they store poorly and are better rebuilt fresh on reheat. State in the workflow: at which step the reserve is pulled, that it is pulled unfinished, how it is cooled and stored, and the reheat method that rebuilds the finish. Where the finishing sauce or dressing is also needed later, split it into an explicit leftover-reserve portion at the point of making it. Reserved portions require their own holding-chain row and a food-safety cooling CCP where a hot item is being stored.

**Sides scope for planned leftovers:** Scale only the components whose leftover value is real. Do not reflexively scale sides that degrade within a day (dressed salads, crisped or charred items) — state that they remain sized for the served meal and why.

**Final plating window:** Order of plating, time available between first plate and service, what fails first if the window is missed.

**Timeline format:** T-minus sequence — each row: "T-XX: [action]." Prose, unordered lists, and non-chronological tables do not satisfy this requirement. For any step with a defined duration, hold window, or maximum tolerance, state the actual duration inline alongside the T-minus reference. Example: "T-20: Steak exits oven — rest uncovered 13 min (sear at T-7)." This allows the cook to track individual step durations independently if service time shifts — the T-minus locates the step in the sequence; the inline duration makes it self-contained.

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

**Format:** A cross-tabbed table with one column per dish and a Total column. Rows are grouped by section. This makes at-a-glance cross-referencing possible when shopping and confirms that quantities have been correctly consolidated across dishes.

```
| Ingredient | Total | [Dish A] | [Dish B] | [Dish N] |
|---|---|---|---|---|
| **SECTION HEADER** | | | | |
| Item *(pantry check)* | consolidated qty | dish qty or — | dish qty or — | — |
```

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

**Sourcing notes:** After the table, add a brief notes block for any item where: (a) a single purchase non-obviously covers multiple uses (e.g., one parsley bunch covering salad, braise, and garnish), (b) a substitution is relevant, or (c) a quantity requires sourcing clarification. Omit the notes block if nothing material to flag.

Do not include rice on the shopping list — rice type is confirmed separately before the meal is finalised.

---

## Cooking Sheet Format (Multi-Component Meals)

The printed cooking sheet for multi-component meals follows the per-dish format and governance rules in `recipe_governance_core.md` Section XI-B, with the additions and compressions below specific to multi-component output.

**Default format: HTML.** Full layout, typography, color, and print CSS specifications are in `recipe_output_html_protocol.md`. Apply that file for all cooking sheet generation. Word (.docx) remains available on explicit request.

---

### Page Order

1. Title + menu overview (bullet list of dishes)
2. T-minus timeline + workflow notes
3. Individual dishes in service order

---

### Shopping List Placement

The cross-tab shopping list (per the Shopping List section above) appears immediately after the menu bullet list, before the T-minus timeline. It is the first substantive content on page 1 and the only point in the document where shopping and execution information overlap.

---

### Meal Workflow Compression

The full Meal Workflow as defined above is the source of truth for planning and review. The printed cooking sheet compresses it as follows:

**Retained in full:**
- T-minus timeline — extended to absorb the Final Plating Window. Plating steps (T-30 through T) are added as the final rows of the T-minus table. A one-line italic note after the table names the first failure if the plating window is missed.
- Holding Chain — retained as a table (Component · Hold Strategy · Max Hold · Drift · Corrective).

**Compressed to two short paragraphs under a single "Workflow Notes" block:**
- Critical path statement
- Failure prioritization (what yields to what)
- Oven sequence (temperatures in order, transition time, which component waits)
- Parallel/sequential summary

**Omitted from the cooking sheet (present in the planning document only):**
- Heat Resource Allocation table — key content folded into Workflow Notes
- Oven Sequencing numbered list — folded into Workflow Notes
- Parallel vs Sequential section — captured by T-minus structure
- Failure Prioritization section — condensed into Workflow Notes
- Final Plating Window section — absorbed into T-minus last rows

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
