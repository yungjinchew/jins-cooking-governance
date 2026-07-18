# Recipe Output — HTML Cooking Sheet Protocol (v1.2)

> Apply when the user requests a cooking sheet, print version, or an HTML export.
> For Word documents — including any bare "generate the document" request — use `recipe_output_docx_protocol.md` instead. See `recipe_governance_core.md` Section XI-B for the format-trigger table.

---

## When to Use This Format

Generate an HTML cooking sheet when:
- The user asks for a cooking sheet, print version, or exported document
- Phrases used: "generate the cooking sheet," "export to HTML," "printable version"

Do **not** generate automatically after producing a recipe or meal in chat. Explicit instruction required — same rule as docx.

---

## Why HTML Over docx

| Factor | HTML | docx |
|---|---|---|
| Token cost | Low — single `create_file` call | High — pip install + Python script + bash + present_files |
| Tool calls | 1 | 5–6 |
| Render | Inline in Claude UI | Download only |
| Print | Browser File → Print | Requires Word or compatible app |
| Styling control | Full CSS + @media print | Limited via python-docx |

---

## Source of Truth

The Notion recipe page (or the recipe generated in the current conversation) is the authoritative source. Pull from it directly — do not reconstruct from memory.

If a Notion page exists: fetch it first, then generate the HTML from its content.

---

## File Naming and Output

- Path: `/mnt/user-data/outputs/[dish-slug]-cooking-sheet.html`
- Example: `chicken-stew-cooking-sheet.html`
- Single self-contained file — no external assets except Google Fonts CDN link

---

## Page Structure

One `.page` div per printed page. Page count by content type:

| Content | Pages |
|---|---|
| Single dish | 1 page |
| Multi-component menu | Page 1: menu overview + shared prep + shopping list; Page 2: T-minus timeline + workflow notes + holding chain; subsequent pages: individual dishes |
| Short dishes (salad, bread, condiment) | Share a page, separated by `<hr class="dish-div">` |
| Long dishes (braise, stew, multi-phase) | Own page |

**Multi-component overflow note:** For a four-dish menu, the shopping list and shared prep block typically fill Page 1 on their own; the T-minus timeline, workflow notes paragraph, and holding chain table go on Page 2. Do not compress all of this onto Page 1 — the T-minus table alone requires significant vertical space for a 15–18 row sequence.

---

## Typography

| Element | Font | Size | Weight | Color |
|---|---|---|---|---|
| Dish name (H1) | Fraunces (serif) | 15pt | 700 | `#1B5E7B` (teal) |
| Section headers (H2) | IBM Plex Sans | 7.5pt | 600 | `#333333` — uppercase, letter-spaced |
| Phase headers (H3) | IBM Plex Sans | 8pt | 600 | `#1B5E7B` (teal) |
| Body / steps | IBM Plex Sans | 8.5pt | 400 | `#222222` |
| Table cells | IBM Plex Sans | 8pt | 400 | `#222222` |
| Table headers | IBM Plex Sans | 7.5pt | 600 | `#333333` |
| Timing subtitle | IBM Plex Sans | 8pt | 400 | `#555555` italic |
| Sensory cues | IBM Plex Sans | 8.5pt | 400 | `#555555` italic |
| Adjust note | IBM Plex Sans | 8pt | 400 | `#555555` italic |
| Page header/footer | IBM Plex Sans | 7–7.5pt | — | `#999` / teal |

Google Fonts import string:
```html
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:ital,wght@0,400;0,500;0,600;1,400&family=Fraunces:opsz,wght@9..144,700&display=swap" rel="stylesheet">
```

---

## Color Palette

```css
--teal:       #1B5E7B   /* dish names, teal accents, timeline time column */
--teal-light: #D0E8F0   /* table header row fills */
--gray-dark:  #333333   /* section headers, table header text */
--gray-mid:   #555555   /* subtitles, cues, adjust notes */
--gray-light: #F0F0F0   /* ingredient table section divider rows */
--crit-bg:    #FEE2E2   /* CCP Critical row */
--major-bg:   #FEF3C7   /* CCP Major row */
--minor-bg:   #F3F4F6   /* CCP Minor row */
```

---

## Page Layout

```
A4 simulation (screen):   width: 210mm; min-height: 297mm
Padding:                  13mm top · 14mm left/right · 16mm bottom
Page shell:               white background, box-shadow for screen preview
Gap between pages:        24px, background #d8d8d8
```

**Running header** (every page): meal name + date (left, teal uppercase) · "Page N" (right, gray)

Page header `margin-bottom`: **7mm**. Do not use 9mm — it consumes too much usable page height, particularly on content-dense pages.

**Footer** (every page): date generated, bottom-right, 7pt gray

---

## Table Rendering Rules (All Tables)

**These rules apply to every table in the cooking sheet without exception.**

### 1. Always use `table-layout: fixed`

```css
table { table-layout: fixed; width: 100%; border-collapse: collapse; }
```

Without `table-layout: fixed`, browsers distribute column widths based on content. This causes narrow columns to collapse on dense content and over-expand on sparse content — producing unpredictable, hard-to-read layouts. Fixed layout respects the specified widths and renders consistently.

### 2. Always add `word-wrap: break-word` to `td`

```css
td { word-wrap: break-word; }
```

Fixed-layout tables do not reflow overflowing text by default. Without `word-wrap: break-word`, long words or phrases overflow their cells. This is especially visible in CCP and holding chain tables where failure mode descriptions are dense.

### 3. Column width specification

In fixed-layout tables, specify widths on `th` and `td` using `nth-child` selectors. Do not rely on `width` attributes in HTML — use CSS only. Where a fixed pixel width is used for one column (e.g., the Severity or Time column), the remaining columns share the remainder by percentage. Percentages sum to slightly more than 100% minus the fixed column when using this mixed approach; browsers with fixed layout distribute proportionally and the result renders correctly.

---

## Per-Dish Layout

In order:
1. `<h1>` dish name (teal, Fraunces)
2. Timing subtitle — italic, one line: yield · total time · active time · oven temp where applicable
3. Dish overview — one sentence, flavor and texture target only (omit trade-off text)
4. Ingredients table
5. Recipe sections (Prep, Active Cooking phases, Holding & Assembly)
6. CCPs table
7. Adjust note (if applicable)

**Overview omission rule:** Very simple supporting dishes (plain bread, simple salad where the title is self-explanatory) may omit the overview.

---

## Ingredient Table

Two columns: Ingredient | Amount. Section divider rows (e.g., Stew / Finish & Gremolata) in light gray with uppercase small label.

```css
.ing-table { table-layout: fixed; }
.ing-table td:first-child { width: 67%; }
.ing-table td:last-child  { width: 33%; font-variant-numeric: tabular-nums; word-wrap: break-word; }

tr.ing-section td {
  background: var(--gray-light);
  font-size: 7pt;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--gray-mid);
  font-weight: 600;
  padding: 3px 6px;
}
```

---

## CCP Table

Three columns only (cooking sheet format — Outcome column omitted):

| Severity | Failure Mode & Cue | Recovery |

Sensory cue folded into Failure Mode column as a combined description.

Row background by severity class:
- `.crit`  → `#FEE2E2`
- `.major` → `#FEF3C7`
- `.minor` → `#F3F4F6`

```css
.ccp-table { table-layout: fixed; }

.ccp-table th:nth-child(1),
.ccp-table td:nth-child(1) {
  width: 58px;
  font-size: 7pt;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  font-weight: 600;
  white-space: nowrap;
}

.ccp-table th:nth-child(2),
.ccp-table td:nth-child(2) { width: 57%; }

.ccp-table th:nth-child(3),
.ccp-table td:nth-child(3) { width: 37%; }

.ccp-table td { font-size: 7.5pt; word-wrap: break-word; }

tr.crit  td { background: var(--crit-bg); }
tr.major td { background: var(--major-bg); }
tr.minor td { background: var(--minor-bg); }
```

**Column proportion note:** The 57% + 37% deliberately sum to 94% of total table width. The remaining ~6% (after the 58px fixed Severity column) is absorbed naturally under fixed layout, producing an approximately 60/40 split of the non-Severity space. Do not try to make the percentages sum to exactly 100% — that produces equal columns and causes the Failure Mode column to be too narrow.

---

## Adjust Note

Not a table. Single italic paragraph immediately after Holding & Assembly. Prefix: *Adjust:*

All three adjustment categories (salt, texture, richness or dish-appropriate equivalents) are folded into a single paragraph. No bullet points, no sub-headers.

---

## T-Minus Timeline (Multi-Component Meals)

Rendered as a two-column table: Time | Action.

```css
.tmin-table { table-layout: fixed; }

.tmin-table th:nth-child(1),
.tmin-table td:first-child {
  width: 100px;
  font-weight: 600;
  color: var(--teal);
  white-space: nowrap;
  font-size: 8pt;
}

.tmin-table td:last-child { word-wrap: break-word; }
```

Immediately below the table: a workflow notes block (8pt, `var(--gray-mid)`, italic) containing two short paragraphs:

1. **Critical path** (bolded label) — the single sequence governing final plate timing; what to sacrifice if it slips
2. **Oven sequencing** (bolded label) — temperatures in order, transition time required between phases, which component holds and how during oven transitions; heat resource allocation by burner for induction-dependent workflows

Wrap the two paragraphs in a `.wf-notes` block:

```css
.wf-notes { font-size: 8pt; color: var(--gray-mid); font-style: italic; line-height: 1.55; margin-bottom: 8px; }
.wf-notes p { margin-bottom: 4px; }
.wf-notes strong { font-style: normal; color: var(--gray-dark); }
```

---

## Holding Chain Table (Multi-Component Meals)

Required for every multi-component cooking sheet. Five columns: Component · Hold Strategy · Max Hold · Drift Risk · Corrective.

```css
.hold-table { table-layout: fixed; font-size: 7.5pt; }
.hold-table th { font-size: 7pt; }
.hold-table td { font-size: 7.5pt; word-wrap: break-word; }

.hold-table th:nth-child(1), .hold-table td:nth-child(1) { width: 14%; }
.hold-table th:nth-child(2), .hold-table td:nth-child(2) { width: 23%; }
.hold-table th:nth-child(3), .hold-table td:nth-child(3) { width: 10%; }
.hold-table th:nth-child(4), .hold-table td:nth-child(4) { width: 17%; }
.hold-table th:nth-child(5), .hold-table td:nth-child(5) { width: 36%; }
```

The Holding Chain table appears on the same page as the T-minus timeline, immediately below the workflow notes block.

---

## Shopping List Table (Multi-Component Meals)

Cross-tab format: Ingredient · Total · [one column per dish].

```css
.shop-table { table-layout: fixed; font-size: 7.5pt; }
.shop-table th { font-size: 7pt; }
.shop-table td { font-size: 7pt; word-wrap: break-word; padding: 3px 5px; }

tr.shop-sec td {
  background: #e4e4e4;
  font-size: 7pt;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #444;
  padding: 3px 5px;
}
```

**Column widths for a four-dish menu** (set via `style=""` on `<th>`):

| Column | Width |
|---|---|
| Ingredient | 30% |
| Total | 15% |
| Dish 1 (lightest) | 14% |
| Dish 2 (main) | 13% |
| Dish 3 | 15% |
| Dish 4 (shortest) | 13% |

Scale proportionally for three-dish or five-dish menus. The Ingredient column should not fall below 28%. Dish columns with minimal content may be as narrow as 12%.

Section divider rows span all columns (`colspan` = total dish count + 2) and use the `.shop-sec` style above.

The shopping list appears on Page 1, after the menu overview and shared prep block, before the T-minus timeline.

---

## Sections Omitted in Cooking Sheet

Per governance (`recipe_governance_core.md` Section XI-B):

- **Flavor Architecture** — governance artifact, not needed for execution
- **Scaling Notes** — cook executes recipe as written
- **Trade-off text** in Dish Overview — compress to one sentence

---

## Sensory Cues

Inline within steps, wrapped in `<span class="cue">`. Rendered italic gray to distinguish from action text. Not extracted to a separate block.

```css
.cue { font-style: italic; color: var(--gray-mid); }
```

---

## Workflow Summary Strip (Active Cooking)

Where the Active Cooking section involves a non-obvious sequencing dependency (e.g., two components sharing the same pan sequentially), add a brief italic workflow summary before the numbered steps. This is a light-background strip with a teal left border — not a section header.

```css
.wf-summary {
  font-size: 7.5pt;
  color: var(--gray-mid);
  font-style: italic;
  background: #f7fbfd;
  border-left: 2.5px solid var(--teal-light);
  padding: 4px 8px;
  margin-bottom: 5px;
  line-height: 1.45;
}
```

---

## Print CSS

```css
@media print {
  body { background: white; padding: 0; }

  .page {
    width: 100%;
    min-height: auto;
    margin: 0;
    padding: 13mm 14mm 18mm;
    box-shadow: none;
    page-break-after: always;
  }

  .page:last-child { page-break-after: avoid; }

  @page { size: A4; margin: 0; }
}
```

User prints via browser: File → Print (desktop) or Share → Print (mobile).

---

## Date Handling

Hardcode the current date in:
- Running page header
- Page footer
- File name slug (optional)

Format: DD Month YYYY (e.g., 26 May 2026).

---

## Governance Compliance Checklist (Internal — Do Not Output)

- [ ] Source fetched from Notion or current conversation — not reconstructed from memory
- [ ] All required sections present per dish: dish name · timing · overview (where applicable) · ingredients · method · holding · CCPs · adjust note
- [ ] Flavor Architecture omitted
- [ ] Scaling Notes omitted
- [ ] CCP table: 3 columns only; Outcome column absent; sensory cue folded into Failure Mode
- [ ] CCP column proportions applied: Severity 58px fixed, Failure Mode ~57%, Recovery ~37%
- [ ] Adjust note: single italic paragraph, "Adjust:" prefix, after Holding & Assembly — not a table
- [ ] Short dishes share a page; long dishes own a page
- [ ] Multi-component Page 1: menu overview + shared prep + shopping list
- [ ] Multi-component Page 2: T-minus timeline + workflow notes (2 italic paragraphs) + holding chain table
- [ ] T-minus timeline on dedicated page — do not compress onto Page 1 with shopping list for 4+ dish menus
- [ ] Workflow notes block present immediately below T-minus: critical path paragraph + oven sequencing paragraph
- [ ] Holding Chain table present for multi-component meals: 5 columns with specified proportions
- [ ] Shopping List table: table-layout fixed; 7pt cells; section divider rows with colspan
- [ ] **All tables use `table-layout: fixed`**
- [ ] **All `td` include `word-wrap: break-word`**
- [ ] Page header `margin-bottom: 7mm` (not 9mm)
- [ ] @media print CSS present; page-break-after: always on .page; last-child exempted
- [ ] Google Fonts CDN link present (IBM Plex Sans + Fraunces)
- [ ] Color palette matches spec
- [ ] Date hardcoded in header and footer
- [ ] Page structure verified on render — every `.page` div carries real content; no empty pages produced by over-eager page assignment
- [ ] No literal markdown characters in rendered output (apply emphasis via HTML/CSS, not `**`/`*`)
