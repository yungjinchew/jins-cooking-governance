# Recipe Output — Print Protocol (v1.3, 2026-10-31)

> One method for every printed cooking sheet. The jins-kitchen print view implements it (`jins-kitchen.netlify.app/#/print/meal/<slug>` and `/#/print/dish/<slug>`; code in `jins-kitchen/tools/print-spec/print_region.js`). A Word file follows the same content rules and page plan. Load this file to print, to change the print view, or to build any printed sheet.
>
> **v1.3:** stored renders made before print spec 1.0 are an archive, not linked in the app. **v1.2:** one stored CCP table format; a revised dish is brought up to §7 in the same save. **v1.1:** flagged readings lead the Measure on the day checklist; the shopping page carries purchase units and Before you shop notes. **v1.0:** print spec 1.0 (the print-view audit).

---

## 1. What prints

"Print it", "print the recipe", "printable version", "cooking sheet" → the app's print view. It renders live from jins-kitchen, so it is always the saved version; nothing needs generating. A Word file only when Jin asks for Word (`recipe_output_docx_protocol.md`, which follows §3–§4 here). Stored PDFs are secondary: the Print button always opens the live view, and the print page links the latest stored PDF generated under this protocol. Renders made before print spec 1.0 (Word, PDF and hand-built HTML files dated before 30 Oct 2026) stay in `site/docs` as an archive and are not listed anywhere in the app.

Two modes:
- **Cooking sheet** (default): for the kitchen. §3 and §4.
- **Full record**: every stored section, one dish per page, with the shopping list. For the archive, not the stove.

## 2. Page

| Property | Value |
|---|---|
| Paper | A4; margins 21 mm top, 19 mm sides, 17 mm bottom (content box 172 × 259 mm) |
| Running header | meal (or dish) name · date, left, teal `#1B5E7B` 8 pt bold · "Page N", right, 7.5 pt grey · thin rule below |
| Footer | "Generated <D Month YYYY> · jins-kitchen · print spec <version>", bottom right, 7 pt grey |
| Type | Arial · meal title 16 pt · dish name 13 pt teal · section labels 9.5 pt uppercase · body 9.5 pt, line height 1.35 · tables and notes 8.5 pt |

Never shrink type below these sizes to save a page.

## 3. Cooking sheet content

### 3.1 Cover (meals)
Title · date, pax, service time · the wine line when one is set · Menu (dish and role) · Shared prep · Meal workflow.

The workflow prints without its planning and check sections, which stay in the database: Menu (already on the cover), Failure prioritisation, Heat resource allocation, Oven sequencing, Parallel vs sequential, (Final) plating window, Timeline verification or arithmetic check, Meal balance audit (with meal nutrition), Sourcing notes, Register, Flavour architecture, Scaling. Everything else prints: Critical path, the T-minus timeline, Holding chain, Contingency, Workflow notes and any meal-specific execution note.

T-minus timeline: every table whose first column holds three or more T− times gets a tick-box column and a **Clock** column worked out from the service time. T−75 reads 16:45 for an 18:00 service; "T−57 to T−50" reads 17:03–17:10; a time before midnight reads "−1 day"; rows timed from an event (E+, P+) or labelled by day stay blank.

### 3.2 Dish
In order: name → timing line (the Yield & Timing section as one italic line) → role and equipment → overview (the first sentence of the Dish Overview) → ingredient table → method → CCP table → Adjust note.

- Ingredient table: two columns, 67 / 33. Amounts with a space before the unit (280 g, 45 ml); spoon, cup and count quantities as fractions (¼ tsp, 1½ tbsp, ½); no "piece"; the alternative amount only when it adds information. Buying spec, prep note and "optional" beside the name.
- Left out of the method: Dish Overview and Yield & Timing (lifted into the header), Flavour architecture, Nutrition estimate, Scaling notes, superseded variants and corrections, and the method's own ingredient list when the ingredient table prints.
- CCP tables print without an Outcome column. Tables stored as raw HTML get the same bold and severity colours as markdown tables.

### 3.3 Measure on the day
One checklist per sheet: the parameters still open in `designed_unvalidated` (a confirmed cook row closes one, the same rule as the `open_validations` view), grouped by dish, as tick · designed value · Observed (blank, to write in). When any open parameter on the sheet is marked `measure: true` (Core, Designed ≠ tested: at most five per menu), the checklist lists those, and one line counts the others ("N other open checks: see Log the cook in the app"). With none marked, it lists every open parameter. It is what the post-cook log needs (data protocol §5). A dish print carries its own at the end of the dish.

### 3.4 Shopping list
Off by default; the phone list is the working list. When added on the print page it prints first, on its own page: the meal's Before you shop notes (`meal.shopping_notes`) above it, then tick · ingredient with its buying spec and any "not …" · what to buy · used in (by dish), with pantry checks on one line underneath. The what-to-buy cell shows the purchase unit (`buy_note`) when one dish uses the item, or the total with "per dish: …" when several do. A dish print never carries a shopping list; its ingredient table is the list.

## 4. Page plan

- The cover flows from the title through the workflow with no forced break.
- A main or a soup starts a page.
- Every other dish (starter, side, sauce, salad, bread, dessert) follows on and is kept whole; when it does not fit in the space left, it moves to the next page.
- With the shopping list added, it is page 1 and the cover starts page 2.
- The Measure on the day checklist goes where it costs less paper: at the end of the cover, beside the timeline, or after the last dish. Fewer pages decides; then fewer pages under 15% full; a tie keeps it on the cover.
- A table repeats its header row on a new page; a row never splits; a heading never ends a page.

## 5. Verification

Before saying a print change is done (a change to the print code or to this file):
1. Open `jins-kitchen.netlify.app/#/printcheck` in Chrome. It pages every meal as A4 with the print stylesheet, using Chrome's own page-breaking (one column per 172 × 259 mm page; equal to a real A4 PDF on page count and dish start pages when checked on 30 Oct 2026), and flags: a page under 15% full · a main not starting a page · a design-only section printed · markdown left in the text · a table wider than the page.
2. Record the totals in `Cooking/MEMORY.md`. Baseline 30 Oct 2026, print spec 1.0: 32 meals, 227 pages (300 before), 20 pages under 40% full; the two pages under 15% full are recipe tails on the data clean-up list.
3. Print one meal to PDF and look at every page.

A flag that comes from a recipe's own text goes on the data clean-up list; a flag that comes from the code is fixed before the change ships.

## 6. Change control

The print code and this file change together and carry the same version: `PRINT.spec` in `print_region.js`, which every page prints in its footer. Minor version for a content or layout rule, major version for a new mode. The code is applied with `python3 tools/print-spec/apply_print_spec.py site/index.html tools/print-spec/print_region.js`, which replaces only the print section and can be re-run on top of other work in `index.html`.

## 7. Authoring rules that keep prints clean

- Method headings use Core's names: Dish Overview, Yield & Timing, Flavor Architecture, Prep Phase, Active Cooking, Holding & Assembly, Critical Control Points, Adjustment Notes, Scaling Notes. No emoji headings.
- The Dish Overview leads with one sentence that states the target; Yield & Timing is one line (yield · total · active · oven where used).
- No ingredient list inside the method when ingredient rows are saved.
- Critical Control Points as a markdown table, `Class | Failure mode | Sensory cue | Recovery`, with the outcome written into the failure-mode cell (Core §VIII).
- A dish revised for any reason is brought up to these rules in the same save: Core headings, the one-sentence overview, the one-line Yield & Timing, the CCP table format, no ingredient list in the method. The revision shows these changes with the rest; dishes are not rewritten in bulk.
- The workflow's Critical Path names what yields first if the timeline slips, because the printed sheet carries no failure-prioritisation section; plating steps are the last T-minus rows.
- Ingredient names keep their proper nouns (Granny Smith, Diamond Crystal, Maris Piper, MB5+).
