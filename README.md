# Recipe Governance Framework

Canonical source for the cooking governance framework. Read by Claude AI projects, Claude Cowork, and ChatGPT.

**GitHub is canonical for file contents. Notion is the control surface for changes.**

- **Files** live here as markdown, fetched by raw URL
- **Proposed changes** live in the Notion [Governance Change Queue](https://app.notion.com/p/466bfaf3d8ed43dfa0e6a4c28caae2ef)
- **Cowork** writes, and so does any Claude session linked to Jin's computer (it counts as Cowork): the scheduled drain applies approved queue rows, and a linked session applies a change Jin approves in that session directly, recording the row as Applied (see `apply_queue.md`)

Notion does not store the files themselves. A fidelity test (18 July 2026) showed its markdown round-trip corrupts them: fence collisions break block structure, brackets get escaped, blank lines are dropped, and filenames auto-link.

---

## Files

| File | Load when |
|---|---|
| `recipe_governance_core.md` | Every recipe, dish, or cooking-method request |
| `recipe_governance_meal_workflow.md` | Two or more components served together |
| `recipe_governance_vault_signals.md` | A Vault digest is active, or Vault retrieval shapes a recipe |
| `recipe_governance_vault_ops.md` | Creating, updating, or preparing a Vault entry |
| `recipe_output_docx_protocol.md` | Generating a Word document (incl. bare "generate the document") |
| `recipe_output_print_protocol.md` | Printing a meal or dish, changing the app's print view, or building any printed sheet (Word or HTML) |
| `recipe_output_html_protocol.md` | HTML named ("export to HTML"): points to the app's print view |
| `recipe_review_protocol.md` | Every complete recipe or menu, before it is declared stable |
| `recipe_output_data_protocol.md` | Recipe stable / Menu stable, save for next time, a document build, a cook log |
| `wine_pairing_cooking_reference.md` | Wine, beverage, cellar, or pairing recommendation |

Current versions live in `manifest.json` only, so this table cannot fall behind.

`manifest.json` lists the same set machine-readably — fetch it first to discover current versions. Raw base: `https://raw.githubusercontent.com/yungjinchew/jins-cooking-governance/refs/heads/main/` (the short `/main/` form can serve a cached copy).

---

## Loading discipline

Load only what the task needs. `recipe_governance_core.md` is always loaded; the rest are conditional. Loading everything wastes context and dilutes the rules that matter for the task at hand.

---

## Proposing a change

Do not edit these files directly. Add a row to the Notion queue with:

- Target file, section, and operation (add / replace / delete / version-bump)
- **Anchor Text** — exact existing text (required for replace and delete)
- **New Text** — verbatim content
- **Rationale** — the triggering failure and why it meets the recurring/generalizable threshold

Jin approves. Cowork applies — on the scheduled drain, or straight away in a session linked to Jin's computer. See `apply_queue.md` for the full procedure.

**Threshold:** rules are codified only when a failure is recurring or generalizable. Single occurrences are noted and deferred. Documentation inconsistencies are corrected inline without a queue row or version bump.
