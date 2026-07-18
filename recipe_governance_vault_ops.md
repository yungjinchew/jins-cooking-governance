# Recipe Governance — Vault Operations (v3.4)

> Load only when creating, updating, or preparing a Recipe Vault entry. Not needed for recipe generation — use `recipe_governance_vault_signals.md` for that.

---

### Digest Format

Maximum 10 entries / 300 words. Include optional fields only where they add signal.

```
VAULT DIGEST — [Date]
Relevant to: [protein / cuisine / occasion]

[Dish Name] | Rating: X | Execution: X | Repeat?: [Yes/With Modifications/No] | Last Made: [date] | Times Made: X
Cook Notes: [1–2 sentences]
Tags: [richness / technique / spice / acid profile — optional]
Served with: [sides / sauce / wine — optional]
Key success: [1 phrase — optional]
Key failure: [1 phrase — optional]
Modification requested: [if Repeat? = With Modifications — required]

Household constraints active: [standing exclusions or preferences not captured above]
```

---

### Post-Cook Update (Required After Each Cook)

Update the Vault entry with:
- **Last Made:** date cooked
- **Times Made:** increment by 1
- **Rating:** 0–10
- **Execution Rating:** 0–10 (10 = flawless)
- **Repeat?:** set or revise; if With Modifications, state the condition (e.g., "guests only," "repeat with leaner side")
- **Cook Notes:** 2–3 sentences — what performed as intended, what was adjusted during cooking, what to change next time. **Language:** use precise, measured language. Do not use casual cooking vocabulary ("fired well," "nailed it," "crushed it") or vague impressions ("tasted amazing"). Describe outcomes and adjustments in concrete, specific terms that will be useful on the next cook.
- **Actual deviations:** material substitutions, timing changes, ingredient deviations — ratings are only useful if tied to what was actually cooked
- **Household split:** if the dish divided opinion, note it — prevents a split verdict being read as consensus
- **Wine pairing outcome:** optional; record if pairing worked, was off, or needs revisiting
- **Designed-parameter confirmation:** where the recipe carried a parameter labeled *planned* rather than *validated* (hold windows, transition durations, recovery times — see Core "Designed ≠ tested"), record what actually happened. Confirmed → the parameter may be relabeled validated in the recipe. Failed or drifted → record the observed behavior and revise. This is the mechanism by which engineered assumptions become evidence; a designed parameter that is never checked against a real cook stays designed indefinitely.

A recipe without post-cook data is an incomplete record. Update manually or via Notion integration.

---

### Pre-Cook Entry

When creating an entry for a dish not yet cooked: Times Made = 0; Rating, Execution Rating, and Repeat? left blank. Include a Post-Cook Update section in the page body listing all required fields above. Do not set Repeat? before the first cook — a pre-cook verdict has no evidential basis.

---

### Schema Gap Handling

Where a dish's Cuisine, Category, Protein, or other field does not fit existing schema options, leave blank rather than force-fit an inaccurate tag. Record the gap in Cook Notes as a signal to expand the schema.
