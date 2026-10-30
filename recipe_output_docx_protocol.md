# Recipe Output — Word (.docx) Cooking Sheet Protocol (v1.1)

> **v1.1 (2026-10-30):** content and page plan follow `recipe_output_print_protocol.md`, so a Word file and the app's print view carry the same sheet.
>
> Content rules and page plan: `recipe_output_print_protocol.md` §3–§4. This file adds what is specific to Word. Apply when the user requests a Word document, a "document," or an export to Word.
> Governance rules that are format-independent live in `recipe_governance_core.md` Section XI-B and govern this file.

---

## When to Use This Format

Generate a Word document when the user explicitly instructs it:

- "generate the Word doc," "export to Word," "generate the document," "make me a docx"

**"Generate the document" means docx, not HTML.** Any document request, regardless of exact wording, defaults to docx unless HTML is explicitly named. No HTML is built: "print it", "cooking sheet" and "printable version" open the app's print view, and "export to HTML" saves that view from the browser (print protocol §1).

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
| Margins | 21 mm top, 19 mm sides, 17 mm bottom (1191 / 1077 / 964 DXA) |
| Content width | 172 mm (9751 DXA) |
| Font | Arial throughout |

**Type scale:** print protocol §2 — meal title 16 pt; dish name 13 pt teal; section labels 9.5 pt uppercase; body 9.5 pt, line height 1.35; tables and notes 8.5 pt; running header 8 pt bold teal with the page number 7.5 pt grey; footer 7 pt grey. Never shrink type below these sizes to save a page.

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

Apply wherever the page plan (print protocol §4) starts a page. Implement by giving heading helpers (`dishName`, `h2`) an optional `pageBreakBefore` parameter.

### 2. Verify page structure before delivery

After building, convert to PDF and confirm **every page carries real content**:

```bash
soffice --headless --convert-to pdf FILE.docx
for p in $(seq 1 N); do pdftotext -f $p -l $p FILE.pdf - | grep -v '^$' \
  | grep -v 'Page\|Generated\|<header text>' | head -1; done
```

Any page whose only content is the running header/footer is a blank-page defect — fix before delivering. A visual check of rasterised pages (`pdftoppm -jpeg -r 100`) is the complement, not the substitute.

### 3. Section page assignments (multi-component meals)

Follow print protocol §4: the cover (title, menu, shared prep, workflow) flows from page 1; `pageBreakBefore` on the name of each main or soup; every other dish `keepNext` / `keepLines` so it stays whole; the shopping list only when Jin asks, on page 1, with the cover from page 2. A long dish across two pages is expected; a blank page is a defect.

### 4. Footer page numbering

Use live Word fields (`PageNumber.CURRENT`) in the header, not hardcoded numbers, so pagination survives editing.

---

## Per-Dish Layout

In order:

Print protocol §3.2, in that order.

---

## CCP Table (docx)

Class | Failure mode | Sensory cue | Recovery, widths 10 / 34 / 26 / 30 %, row fill by severity (print protocol §3.2). Class cell bold.

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

Print protocol §3.1 and §4: the cover (title, menu, shared prep, the workflow as printed) flows from page 1; the Measure on the day checklist where the print protocol places it; the shopping list only when asked, on page 1, with the cover from page 2; dishes in service order, each main or soup starting a page.

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
- [ ] A4, margins 21 / 19 / 17 mm, Arial, type scale per print protocol §2
- [ ] Color system applied: teal dish names, gray section headers, teal-light table headers
- [ ] Header on every page (name + date left, live page number right); footer with generation date
- [ ] **All section/dish breaks via `pageBreakBefore` on the heading — no standalone break paragraphs**
- [ ] **PDF rendered and every page confirmed to carry real content — zero blank pages**
- [ ] Page plan per print protocol §4: mains and soups start a page; other dishes whole; shopping list only when asked
- [ ] Per-dish sequence per print protocol §3.2
- [ ] CCP table: Class · Failure mode · Sensory cue · Recovery with severity fills; no Outcome column
- [ ] Adjustment Notes rendered as a single italic "Adjust:" paragraph, not a table
- [ ] Flavor Architecture and Scaling Notes omitted per Core XI-B
- [ ] No literal markdown characters anywhere in the rendered output
- [ ] Table header rows set `tableHeader: true`; column widths explicit
- [ ] File in `/mnt/user-data/outputs/` and presented via `present_files`
