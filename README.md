# Recipe Governance Framework

Canonical source for the cooking governance framework. Read by Claude AI projects, Claude Cowork, and ChatGPT.

**GitHub is canonical for file contents. Notion is the control surface for changes.**

- **Files** live here as markdown, fetched by raw URL
- **Proposed changes** live in the Notion [Governance Change Queue](https://app.notion.com/p/466bfaf3d8ed43dfa0e6a4c28caae2ef)
- **Cowork** is the only writer — it drains approved queue rows and commits (see `apply_queue.md`)

Notion does not store the files themselves. A fidelity test (18 July 2026) showed its markdown round-trip corrupts them: fence collisions break block structure, brackets get escaped, blank lines are dropped, and filenames auto-link.

---

## Files

| File | Version | Load when |
|---|---|---|
| `recipe_governance_core.md` | v3.4 | Every recipe, dish, or cooking-method request |
| `recipe_governance_meal_workflow.md` | v3.4 | Two or more components served together |
| `recipe_governance_vault_signals.md` | v3.4 | A Vault digest is active, or Vault retrieval shapes a recipe |
| `recipe_governance_vault_ops.md` | v3.4 | Creating, updating, or preparing a Vault entry |
| `recipe_output_docx_protocol.md` | v1.0 | Generating a Word document (incl. bare "generate the document") |
| `recipe_output_html_protocol.md` | v1.2 | Generating an HTML cooking sheet |
| `wine_pairing_cooking_reference.md` | — | Wine, beverage, cellar, or pairing recommendation |

`manifest.json` lists the same set machine-readably — fetch it first to discover current versions.

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

Jin approves. Cowork applies. See `apply_queue.md` for the full procedure.

**Threshold:** rules are codified only when a failure is recurring or generalizable. Single occurrences are noted and deferred. Documentation inconsistencies are corrected inline without a queue row or version bump.
