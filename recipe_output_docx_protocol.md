# Recipe Output — Word (.docx) Cooking Sheet Protocol (v1.0)

> Companion to `recipe_output_html_protocol.md`. Apply when the user requests a Word document, a "document," or an export to Word.
> Governance rules that are format-independent live in `recipe_governance_core.md` Section XI-B and govern this file.

---

## When to Use This Format

Generate a Word document when the user explicitly instructs it:

- "generate the Word doc," "export to Word," "generate the document," "make me a docx"

**"Generate the document" means docx, not HTML.** Any document request, regardless of exact wording, defaults to docx unless HTML is explicitly named. HTML is produced only on explicit request ("generate the HTML," "cooking sheet," "printable version," "export to HTML").

Do **not** generate automatically after producing a recipe or meal in chat. Explicit instruction required.

---

## Source of Truth

The Notion recipe page, or the recipe as finalised in the current conversation, is authoritative. Pull from it directly — do not reconstruct from memory. If a Notion page exists, fetch it first.

Where a menu has been through a review cycle in-conversation, the **final reconciled version** is the source — not an earlier draft.

---

## File Naming and Output

- Path: `/mnt/user-data/outputs/[meal-or-dish-slug].docx`
- Example: `bbq-ribs-dinner.docx`, `chicken-stew.docx`
- Where a variant materially changes quantities (e.g., a two-rack version), suffix it: `bbq-ribs-dinner-2racks.docx`

---

## Page Setup

| Property | Value |
|---|---|
| Paper | A4 (11906 × 16838 DXA) |
| Margins | 0.75" all sides (1080 DXA) |
| Content width | 9746 DXA |
| Font | Arial throughout |

**Type scale:**

| Element | Size | Weight | Color |
|---|---|---|---|
| Meal title | 15pt | bold | `#1B5E7B` teal |
| Dish name (H1) | 13pt | bold | `#1B5E7B` teal |
| Section header (H2) | 11pt | bold | `#333333` gray |
| Phase header (H3) | 10pt | bold | `#1B5E7B` teal |
| Body / steps | 9.5pt | regular | `#222222` |
| Table cells | 8–9pt | regular | `#222222` |
| Table header row | 7.5–8pt | bold | `#333333` on `#D0E8F0` |
| Timing info row | 9pt | italic | `#555555` on `#EFEFEF` |
| Page header / footer | 7.5pt | — | teal / `#999999` |

**Color system:** teal `#1B5E7B` · gray `#333333` · teal-light `#D0E8F0` (table header fills) · CCP severity fills: Critical `#FEE2E2`, Major `#FEF3C7`, Minor `#F3F4F6`.

**Header (every page):** meal or dish name + date (left, teal, uppercase) · "Page N" (right, gray), with a thin teal-light bottom rule.
**Footer (every page):** "Generated [DD Month YYYY]", bottom-right, gray.

---

## Pagination — Governing Rules

**These rules exist because manual break paragraphs have repeatedly produced blank pages and mis-shared pages. They are not stylistic preferences.**

### 1. Use `pageBreakBefore` on the heading — never a standalone break paragraph

A standalone break paragraph (`new Paragraph({ children: [new PageBreak()] })`) leaves an empty paragraph in the flow. On a short section, that stray paragraph tips the following break onto a fresh empty page, producing a blank page in the output.

```js
// WRONG — leaves a stray empty paragraph, causes blank pages
elements.push(pageBreak());
elements.push(h2('Shopping List'));

// RIGHT — break is a property of the heading itself
elements.push(h2('Shopping List', /* pageBreakBefore */ true));
```

Apply to every section and dish transition. Implement by giving heading helpers (`dishName`, `h2`) an optional `pageBreakBefore` parameter.

### 2. Verify page structure before delivery

After building, convert to PDF and confirm **every page carries real content**:

```bash
soffice --headless --convert-to pdf FILE.docx
for p in $(seq 1 N); do pdftotext -f $p -l $p FILE.pdf - | grep -v '^$' \
  | grep -v 'Page\|Generated\|<header text>' | head -1; done
```

Any page whose only content is the running header/footer is a blank-page defect — fix before delivering. A visual check of rasterised pages (`pdftoppm -jpeg -r 100`) is the complement, not the substitute.

### 3. Section page assignments (multi-component meals)

| Content | Placement |
|---|---|
| Menu overview + Meal Workflow | Page 1 (continues as needed) |
| Shared Prep | Own page — `pageBreakBefore` |
| Shopping List | Own page — `pageBreakBefore` |
| Each dish | Own page — `pageBreakBefore` on the dish name |

Shared Prep and Shopping List must **not** share a page. A long dish spanning two pages is correct and expected; a *blank* page is not.

### 4. Footer page numbering

Use live Word fields (`PageNumber.CURRENT`) in the header, not hardcoded numbers, so pagination survives editing.

---

## Per-Dish Layout

In order:

1. Dish name — H1, teal, 13pt bold, `pageBreakBefore: true` (except the first dish if it opens the document)
2. **Yield & Timing** — a compact shaded info row (single-cell table, `#EFEFEF` fill, italic gray), immediately below the dish name
3. Dish overview — one sentence, flavor and texture target only
4. Ingredients — two-column table (Ingredient | Amount), 67% / 33%, tight rows, right-aligned amounts, section divider rows in `#F0F0F0` with uppercase small label
5. Recipe sections — H2 headers (Prep Phase, Active Cooking, Holding & Assembly), numbered steps with hanging indent, compact spacing
6. CCP table — **four columns** (see below)
7. Adjust note — single italic paragraph, "Adjust:" prefix

---

## CCP Table (docx — four columns)

Unlike the HTML cooking sheet (three columns), the docx retains the Outcome-adjacent detail by splitting cue from failure mode:

| Class | Failure Mode | Sensory Cue | Recovery |
|---|---|---|---|

Approximate widths: 10% · 34% · 26% · 30%. Row fill by severity class (Critical/Major/Minor per the color system). Class cell bold. 8pt cells, 7.5–8pt header.

---

## Tables — General

- Set explicit `columnWidths` matching the summed cell widths; do not rely on auto-layout
- Thin borders, `#CCCCCC`
- Cell margins ~40 DXA vertical, 90 DXA horizontal
- Header rows: `tableHeader: true` so they repeat if a table breaks across pages
- Section divider rows span all columns via `columnSpan`

---

## Markdown Does Not Render

docx has no markdown parser. Emphasis must be applied as `TextRun` properties (`bold`, `italics`), never as literal `**` or `*` characters.

**Common failure:** carrying markdown markers over from a chat draft — e.g., `*(pantry check)*` renders as literal asterisks in the document. Strip all markdown syntax when porting chat content into docx and apply formatting via run properties instead.

---

## Multi-Component Meal Structure

1. **Page 1** — Meal title, menu bullet list, then the Meal Workflow (critical path, failure prioritization, oven sequencing, parallel vs. sequential, holding chain table, T-minus timeline, meal balance audit). Continues to page 2 as needed.
2. **Shared Prep** — own page
3. **Shopping List** — own page, cross-tab format (Ingredient · Total · one column per dish), grouped by section
4. **Dishes** — each on its own page, in service order

---

## Build and Verification Workflow

1. Read this protocol and `recipe_governance_core.md` Section XI-B before writing any build script
2. Build with the `docx` npm library via Node
3. Convert to PDF (`soffice --headless --convert-to pdf`)
4. Rasterise (`pdftoppm -jpeg -r 100`) and **view every page**
5. Run the blank-page text check (Pagination rule 2)
6. Copy to `/mnt/user-data/outputs/` and present

Do not deliver without steps 3–5. Pagination defects are invisible in the build step and only appear on render.

---

## Governance Compliance Checklist (Internal — Do Not Output)

- [ ] Source is the final reconciled recipe — not an earlier draft or memory
- [ ] A4, 0.75" margins, Arial throughout
- [ ] Color system applied: teal dish names, gray section headers, teal-light table headers
- [ ] Header on every page (name + date left, live page number right); footer with generation date
- [ ] **All section/dish breaks via `pageBreakBefore` on the heading — no standalone break paragraphs**
- [ ] **PDF rendered and every page confirmed to carry real content — zero blank pages**
- [ ] Shared Prep and Shopping List each on their own page, not shared
- [ ] Each dish starts on a new page
- [ ] Per-dish sequence: name → shaded timing row → overview → ingredients → sections → CCPs → Adjust note
- [ ] CCP table has four columns (Class · Failure Mode · Sensory Cue · Recovery) with severity fills
- [ ] Adjustment Notes rendered as a single italic "Adjust:" paragraph, not a table
- [ ] Flavor Architecture and Scaling Notes omitted per Core XI-B
- [ ] No literal markdown characters anywhere in the rendered output
- [ ] Table header rows set `tableHeader: true`; column widths explicit
- [ ] File in `/mnt/user-data/outputs/` and presented via `present_files`
