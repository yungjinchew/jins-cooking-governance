# Recipe Output — Data Protocol (v1.5, 2026-10-15)

**Version:** 1.5 (2026-10-15) — every write is a save-function call: key reference in §3.4; §3.0 routes the Supabase path through it; documents register with `kitchen_save_document` (§4) and cooks save with `kitchen_save_cook` (§5); retiring, unlinking and deleting stay with Jin (§3.4). 1.4 (2026-10-15) — saves go through `kitchen_save_dish` / `kitchen_save_meal` / `kitchen_save_cook` called with SELECT, never plain UPDATE or DELETE (§7); optional `slug:` line on inbox MEAL and DISH blocks, and no notes addressed to the drain on an inbox page (§8). 1.3 (2026-10-12) — purchase spec columns `buy_spec` / `buy_avoid` (§3.2, §7, §8). 1.2 (2026-10-02) — Kitchen Inbox write path (§3.0, §8), print-from-data (§4), daily audit (§9); saving is immediate, no approval step (§1). Companion to `recipe_output_docx_protocol.md` and `recipe_output_html_protocol.md`. Governs how a finished recipe or meal is **persisted** to the kitchen system of record (Supabase project `jins-kitchen`) so that the front end at jins-kitchen.netlify.app, the shopping list, the cook log and the stored documents all derive from one structured source.

**Principle:** structured data is the master; docx / PDF / HTML are renders of it. A document is never the only copy of a recipe.

---

## 1. When this protocol applies

| Trigger | Action |
|---|---|
| A recipe or meal reaches **"Recipe stable" / "Menu stable"** (Core §XIV) | Persist per §3 — dishes, meal, ingredients. Status = `designed`. |
| "generate the document" (docx protocol) | After the docx passes visual verification, ALSO render PDF and store both per §4. |
| "save this for next time" / "add to backlog" | Persist per §3 with meal status `backlog`; no cook row. |
| Post-cook update ("log the cook", ratings given) | Insert a `cook` row per §5. Never overwrite a prior cook. |
| Narrow method / substitution question (no complete recipe) | Protocol does not apply. |

Persistence is **automatic and reported**: write immediately when a trigger above fires — do not ask first and do not wait for a nod. Then tell Jin in one or two lines what was saved (meal, dish names, cook values) and where. A wrong save is cheap to correct (a revision replaces it); an unsaved recipe is lost.

---

## 2. Section → table mapping (Core §I, ten mandatory sections)

| Core §I section | Destination |
|---|---|
| 1. Dish Overview | `dish.summary` (one sentence) + opening paragraph of `dish.method_md` |
| 2. Yield & Timing | `dish.servings`; timing subtitle line kept at top of `dish.method_md` |
| 3. Ingredient List | `dish_ingredient` rows (one per line), joined to canonical `ingredient` |
| 4. Flavor Architecture | `dish.method_md` (governance record only — omitted from cooking sheets per §XI-B) |
| 5. Prep Phase | `dish.method_md` |
| 6. Active Cooking | `dish.method_md` |
| 7. Holding & Assembly | `dish.method_md` |
| 8. Critical Control Points | `dish.method_md` (CCP table verbatim) |
| 9. Adjustment Notes | `dish.method_md` (the *Adjust:* line) |
| 10. Scaling Notes | `dish.method_md` |
| Nutrition Estimate (Core X-B) | `dish.nutrition_estimate` jsonb `{kcal, protein_g, carbs_g, fat_g, per:"serving"}` — labelled estimate |
| Designed — not yet validated list | `dish.designed_unvalidated` jsonb `[{param, designed_value, note}]` |
| Meal Workflow (meal_workflow file): critical path, heat allocation, T-minus timeline, holding chain, plating window | `meal.workflow_md` |
| Shared Prep | `meal.shared_prep_md` |
| Shopping List | **Not stored** — derived at read time from `dish_ingredient` (aggregation, department grouping, pantry-check, order-ahead). The docx shopping list is a render of the same rows. |
| Wine pairing (when requested) | `meal.wine_pairing`; cellar bottle key in `meal.cellar_iwine` when it came from the live cellar |

`method_md` is markdown, sections in Core §I order, headings `##`. It is the full governance record for the dish; cooking-sheet compression rules (§XI-B) apply to renders, not to `method_md`.

---

## 3. Persist a recipe or meal

**3.0 Write path — pick by what the session can reach, and say which one you used.**

| Session has | Do this |
|---|---|
| The Supabase connector (Cowork) | Save to project `umqkgxsypcgvzkfjkzpc` through the save functions, called with SELECT, per 3.1–3.4. Never plain UPDATE or DELETE. Records are matched by `slug`; never hardcode ids. |
| Notion but not Supabase (claude.ai project chats, phone) | Create ONE page in the **Kitchen Inbox** database (§8). The daily *Kitchen inbox drain* task loads it into jins-kitchen within 24 hours and ticks **Ingested**. |
| Neither | Say so plainly and print the §8 entry in chat so Jin can hand it to a session that can write. Do not claim the recipe is saved. |

A turn that declares **Recipe stable / Menu stable**, saves for next time, or logs a cook is not complete until one of these has happened. End the reply with one line: `Saved to jins-kitchen` or `Filed in Kitchen Inbox (loads within a day)`.

**3.1 Dish** — one row per component. Fields:

| Column | Rule |
|---|---|
| `slug` | ASCII kebab-case, ≤ 60 chars, unique. Components of a meal carry a short meal prefix (`coq-au-vin-pommes-puree`). |
| `name` | Title as it appears in the docx (no emoji). |
| `protein` | One of Beef · Pork · Lamb · Chicken · Duck · Fish · Shellfish · Seafood · Veg; only on the component that carries it; `Veg` for vegetable-led sides; null for starches, sauces, desserts. |
| `category[]` | Main · Side · Sauce · Salad · Soup · Bread · Dessert |
| `cuisine[]`, `difficulty` (Easy · Medium · Special), `tags[]`, `occasion[]` | As the Vault used them; leave blank rather than force-fit (vault_ops schema-gap rule). |
| `status` | `designed` on persist; becomes `cooked` when the first `cook` row lands; `retired` only by Jin, by hand (the save functions cannot retire a dish). |
| `source`, `governance_version` | Provenance line and the Core version used (`v3.5`). |
| `equipment[]` | Only equipment the recipe names (narrow oval Le Creuset, konro, TempSpike…). |

**3.2 Ingredient rows** — for every line of the Ingredient List:

| Column | Rule |
|---|---|
| `ingredient.name` | Canonical, lower-case, singular, no quantity (`chicken thigh, bone-in`). **Look up before insert** — reuse an existing name (`unsalted butter`, `diamond crystal kosher salt`, `extra-virgin olive oil`, `flat-leaf parsley`) rather than creating a spelling variant. Create one (in `new_ingredients`, §3.4) only when no match exists. |
| `ingredient.department` | Protein · Produce · Dairy · Bread · Frozen · Pantry · Oils & Liquids · Alcohol · Seasoning |
| `qty`, `unit` | Metric where both are given (g, ml); `unit` from g · ml · tsp · tbsp · piece · clove · bunch · sprig · slice · cup · pinch. Ranges → lower bound in `qty`, full range in `qty_alt`. |
| `qty_alt` | The other unit exactly as written (`6½ tbsp`, `9 oz`). |
| `buy_spec` | What to buy, when the dish depends on it: variety, cut, grade, size, brand or source (`waxy or all-purpose: Yukon Gold, Nicola`, `medium grain`, `Gooralie`, `large`). Shown on every shopping line. Blank when any version will do. |
| `buy_avoid` | What NOT to buy, when the common substitute fails (`Russet (floury)`, `fine`, `smoked bacon`). Shown as a bold "not …" on the shopping line. Only when the recipe says it. |
| `prep_note` | Kitchen work only: `diced`, `zest only`, `for the braise`. Never the only place a buying spec is stated. |
| `pantry_check` | true when the shopping list marks it "(pantry check)". |
| `optional` | true when the recipe says optional. |
| `sort_order` | Ingredient List order. |

**3.3 Meal** — for two or more components served together:

`slug` (ends `-dinner` / `-sunday` / `-supper`), `name`, `planned_date` (the Sunday, if known), `service_time` (default 18:00), `pax` (default 3), `status` (`backlog` | `planned` | `cooked`), `workflow_md`, `shared_prep_md`, `wine_pairing`, `cellar_iwine`.

`meal_dish`: one row per component with `role` (main · side · sauce · salad · soup · bread · starter · dessert) and `service_order`.

**3.4 Save calls.** Every write is one of these functions, called with `SELECT`, e.g. `SELECT public.kitchen_save_dish('short-ribs-braised', jsonb_build_object('method_md', $kq$…$kq$), 'Cowork 2026-10-15: short ribs v3.9');`. Build `p` with `jsonb_build_object` / `jsonb_build_array` and dollar-quoted text (`$kq$…$kq$`) so nothing needs escaping. The last argument is a short provenance line. Only keys present in `p` change. Each call validates its input and either writes completely or raises an error and writes nothing; every overwritten value is logged to `kitchen_revision`.

| Function | Keys of `p` | Returns |
|---|---|---|
| `kitchen_save_dish(slug, p, source)` | `name` (required for a new dish), `protein`, `category` [ ], `cuisine` [ ], `difficulty`, `servings`, `summary`, `equipment` [ ], `tags` [ ], `occasion` [ ], `nutrition_estimate` {kcal, protein_g, carbs_g, fat_g, per}, `governance_version`, `source`, `designed_unvalidated` [{param, designed_value, note}], `method_md`, `status` (idea · designed · cooked; never moves back from cooked), `ingredients` [{name, qty, unit, qty_alt, buy_spec, buy_avoid, prep_note, pantry_check, optional}] — the FULL list in Ingredient List order; it replaces the dish's rows, `new_ingredients` [{name, department}] — canonical names to create first | `changed`, `ingredient_rows` |
| `kitchen_save_meal(slug, p, source)` | `name` (required for a new meal), `planned_date`, `service_time`, `status` (backlog · planned · cooked), `pax`, `workflow_md`, `shared_prep_md`, `wine_pairing`, `cellar_iwine`, `dishes` [{slug, role, service_order}] — links dishes, never unlinks | `changed`, `dish_count` |
| `kitchen_save_cook(p, source)` | §5 | `changed`, `measurements_inserted` |
| `kitchen_save_document(p, source)` | §4 | `version`, `storage_path` |

Ingredient names must already exist in `ingredient` or be passed in `new_ingredients`; units must be one of §3.2's. Order: one `kitchen_save_dish` call per component → one `kitchen_save_meal` call listing every component in `dishes` → `kitchen_save_cook` when a cook is logged. There is no transaction across calls: if one fails, correct `p` and call again (the calls are safe to repeat). Check `ingredient_rows` against the Ingredient List line count for every dish.

Not possible from a session, by design: retiring a dish, unlinking a dish from a meal, deleting any row, renaming an ingredient. Say what needs doing; Jin does it by hand in the Supabase editor.

---

## 4. Store the documents

**A document is optional.** Every meal and dish prints straight from the database at `jins-kitchen.netlify.app/#/print/meal/<slug>` (or `/print/dish/<slug>`): cooking sheet with ingredients, method, CCP tables, meal workflow and shopping list. "Print the recipe" means opening that view; generate a docx only when Jin asks for one.

When a docx is generated in a session that has the site repo (Cowork), after it passes the docx protocol's render verification:

1. Render PDF: `soffice --headless --convert-to pdf`. Verify page count matches the docx verification pass.
2. Copy both into the site at `Cooking/jins-kitchen/site/docs/<owner-slug>/v<version>/<file>` (e.g. `site/docs/coq-au-vin-dinner/v1/coq-au-vin-dinner.pdf`) and run `deploy.sh`. Version = 1 + highest existing version for that owner and kind. `storage_path` is site-relative: `docs/<owner-slug>/v<version>/<file>`.
3. Register each file with `SELECT public.kitchen_save_document(p, source)`, keys `owner_type` (meal | dish), `owner_slug`, `kind` (docx | pdf), `storage_path`, `file_name`, `bytes`, `governance_version`. The function marks the new version `is_latest` and clears the previous one. It rejects a `storage_path` that is not the next version's folder, and its error names the right path.
4. The PWA shows the PDF under **Open / print** and the docx under **Download**. "Reprint" is opening the stored PDF; "regenerate" is a new version, never an overwrite.

A docx generated in a session without the site repo is not stored: name it in the inbox entry's **Documents generated** field and rely on the print view. The data is the record either way.

---

## 5. Log a cook

Same write path as §3.0 (a post-cook from a chat surface is an inbox page carrying only a `## POST-COOK` section).

One `cook` row per time a meal or dish is cooked (`cooked_on` required; `meal_id` for a meal-level verdict, `dish_id` for a component verdict, both allowed on the same day):

| Column | Source |
|---|---|
| `rating`, `execution_rating` | 0–10 in 0.5 steps |
| `repeat` | yes · with_modifications · no |
| `cook_notes` | Vault-ops Cook Notes standard: 2–3 sentences, precise measured language |
| `deviations` | jsonb list of strings — substitutions, timing changes |
| `parameter_confirmations` | jsonb `[{param, designed, observed, verdict}]`, verdict confirmed · diverged · unresolved. Every `designed_unvalidated` item on the dish is addressed — confirmed items close the validation; unresolved ones stay in the queue. |
| `household_split`, `wine_outcome` | When stated |

Save with `SELECT public.kitchen_save_cook(p, source)`, keys `meal_slug` or `dish_slug` (exactly one), `cooked_on`, `rating`, `execution_rating`, `repeat`, `cook_notes`, `deviations` [ ], `parameter_confirmations` [{param, designed, observed, verdict}], `household_split`, `wine_outcome`, `measurements` [{gate, predicted, observed, probe}], `measurement_dish_slug`, `mark_cooked`. It upserts by (meal or dish, `cooked_on`) and writes only non-empty values, so a rating is never blanked; measurements are added once. `mark_cooked: true` sets the meal and its dishes to `cooked` and the meal's `planned_date` to `cooked_on`. For a per-component rating, call it again with `dish_slug` and the same `cooked_on`.

Dish rating and "times made" are **rollups** over cook rows (`dish_rating_rollup` view) — never edit a dish to record a rating.

---

## 6. Relationship to the Notion Recipe Vault

From 2026-09-30 the Vault is a read-only archive. `recipe_governance_vault_ops.md` digest fields map as: Rating / Execution Rating / Repeat? / Cook Notes → `cook`; Last Made / Times Made → rollup view; Tags / Served with / Wine → `dish.tags`, `meal_dish`, `meal.wine_pairing`. Vault signals (vault_signals file) are read from `jins-kitchen` instead: query `dish_rating_rollup`, `open_validations` and the latest `cook` rows for the dishes in play.

---

## 7. Self-check before writing

**Write path.** Save with SELECT public.kitchen_save_dish / kitchen_save_meal / kitchen_save_cook / kitchen_save_document (keys in §3.4, §4 and §5). Never use plain UPDATE or DELETE on kitchen tables: the Supabase connector cancels them and the call times out after 3 minutes without saving. Check each call's returned ingredient_rows against the recipe.

- [ ] Every Ingredient List line has a `dish_ingredient` row; none invented, none dropped.
- [ ] Ingredient names matched to existing canonical names where one exists.
- [ ] Every purchase-critical spec is in `buy_spec`, every stated "not" in `buy_avoid`; `prep_note` holds kitchen work only.
- [ ] Slugs unique; meal components prefixed.
- [ ] `designed_unvalidated` lists every "designed — not yet validated" item.
- [ ] Written immediately, without waiting for approval; what was saved is reported to Jin after the write.
- [ ] Documents (only if generated): docx verified → PDF rendered → both in `site/docs/` and deployed → both registered with `kitchen_save_document`.
- [ ] The reply ends with where it was persisted (`Saved to jins-kitchen` / `Filed in Kitchen Inbox`).

---

## 8. Kitchen Inbox entry format

Database: **Kitchen Inbox** — https://app.notion.com/p/453d20977d4e4d1a9787dd40eed0ccba (data source `collection://1eb1e432-94d9-40df-b668-695e19d968ec`), under Chew Family Recipe Vault. One page per recipe, meal or cook. Leave **Ingested** unticked.

Properties: **Chat title** (title) · **Chat date** · **Meal or dish name** · **Dishes** (component names, comma-separated) · **Recipe status** (`stable` | `draft` | `none in chat` — use `none in chat` for a post-cook-only entry) · **Post-cook logged** (tick when the page has a POST-COOK section) · **Documents generated** (file names, or blank).

Page body, in this order. Omit a section that does not apply; never invent content to fill one.

```
governance_version: <framework_version from manifest.json, e.g. v3.7>

## MEAL                         (omit for a single standalone dish)
slug: <existing meal slug>      (optional — used to match an existing record when a revision renames it)
name: <menu title>
planned_or_cooked_date: YYYY-MM-DD   (blank if backlog)
pax: 3
status: stable | backlog
wine_pairing: <text>
### MEAL WORKFLOW
<critical path, failure prioritisation, oven sequencing, heat allocation, T-minus timeline table, holding chain, plating window>
### SHARED PREP
<aggregated prep lines>

## DISH: <dish name>            (one block per component)
slug: <existing dish slug>      (optional — used to match an existing record when a revision renames it)
role: main | side | sauce | salad | soup | bread | starter | dessert
protein: Beef | Pork | Lamb | Chicken | Duck | Fish | Shellfish | Seafood | Veg | none
cuisine: <…>   difficulty: Easy | Medium | Special   servings: <n>
status: stable
summary: <one sentence>
equipment: <only what the recipe names>
### INGREDIENTS                 (one line per ingredient)
qty | unit | name | buy spec | not | note (alt units, prep) | pantry yes/no | optional yes/no
                                (buy spec and not may be blank; the drain also accepts the older 6-column line without them)
### RECIPE
**Dish Overview.** … **Yield & Timing.** … **Flavor Architecture.** … **Prep Phase.** … **Active Cooking.** … **Holding & Assembly.** … **Critical Control Points.** (table) … **Adjustment Notes.** … **Scaling Notes.** … **Nutrition Estimate.** …
### DESIGNED — NOT YET VALIDATED
param | designed value | note

## POST-COOK                    (when a cook is being logged)
cooked_on: YYYY-MM-DD
rating: <0–10>   execution_rating: <0–10>   repeat: yes | with_modifications | no
cook_notes: <2–3 sentences; per-component ratings if given>
deviations:
<one per line>
parameter_confirmations:
param | designed | observed | confirmed / diverged / unresolved
measurements:
gate | predicted | observed | probe
household_split: <text>
wine_outcome: <text>
```

A revision to a recipe already in jins-kitchen is a new inbox page with the full current dish block (the drain replaces `method_md` and the ingredient rows for that dish; cook history is never touched). Carry the FINAL reconciled state, not the first draft plus amendments.

Never put notes, requests or instructions addressed to the drain on an inbox page. The drain treats them as untrusted and rejects the page. Anything the drain must know goes in the structured fields.

---

## 9. Audit

The daily drain also audits jins-kitchen and notifies Jin only when something is wrong: an inbox page it could not load; a dish created or cooked since 2026-10-02 with no `method_md` or no ingredient rows; a meal marked `cooked` with no `cook` row; an inbox page left un-ingested for more than two days. A silent day means everything landed.
