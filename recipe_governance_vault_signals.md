# Recipe Governance — Vault Signals (v3.5)

> Load in addition to `recipe_governance_core.md` when a Vault digest is provided or Notion/Recipe Vault retrieval is being used to shape a recipe. Do not load for Vault entry creation or updates — use `recipe_governance_vault_ops.md` for that.

---

### When Vault Context Is Provided

A Vault digest may be supplied as a preamble to any recipe request. When present:

1. **Read it before generating anything.** It is an input constraint, not background.
2. **Do not reproduce it in the output.** Use it silently to shape the recipe.
3. **Apply all five constraint types below** where relevant data exists.

**Proactive retrieval:** If Notion access is available, retrieve a relevant digest before generating rather than waiting for a manual paste — when doing so would materially affect dish selection or repeat logic.

**Data discipline:** Use only fields provided; do not infer missing ratings, repeat verdicts, or cook notes. One or two entries indicate direction, not a household rule — a small corpus creates a suggestion, not a constraint.

**Conflict hierarchy** — apply in this order:

1. User's explicit current request
2. Safety, doneness, and execution governance (Core Sections IV–VIII)
3. Current ingredient and equipment constraints
4. Strong negative Vault signals (Repeat? = No, execution failures)
5. Strong positive Vault signals (Repeat? = Yes, high-rated repeats)
6. Recency and rotation logic
7. General preference inference

If Vault context conflicts with the user's current request, honor the request but flag the tension briefly. If it conflicts with Core framework rules, the framework prevails without exception.

Safety override: The user's explicit current request controls preference, scope, and desired outcome, but never overrides food-safety, allergen, or execution hard limits. Where a request conflicts with a Core safety rule, the Core rule prevails; state the conflict and the safest executable alternative rather than silently complying or silently changing the request.

---

### When No Digest Is Provided

If no digest accompanies the request and Vault retrieval is unavailable or not materially relevant, proceed under Core framework as normal. Do not invent or assume historical data. Do not note the absence of a digest.

---

### The Five Constraint Types

**1. Preference signals (Repeat? = Yes)**

Confirmed household hits. A single Rating ≥ 8.5 is promising but can be outweighed by contrary Cook Notes. Times Made ≥ 2 with Rating ≥ 8.0 is more reliable — repeated success outranks a one-off high score. When generating in the same protein/cuisine/occasion space:
- Build toward the flavor profile that produced those results.
- If the new request matches a Repeat? = Yes entry's protein and occasion, differentiate deliberately (name the difference in Dish Overview) or acknowledge the variation.
- Do not treat a high rating as permission to repeat the same dish unless explicitly asked.

**2. Conditional signals (Repeat? = With Modifications)**

Recoverable dishes with a known fix. Preserve the successful core; apply the requested modification as a design constraint — do not regenerate the original unchanged. Where the modification affects flavor balance, texture, workflow, or doneness thresholds, reflect it in Flavor Architecture, CCPs, or Meal Workflow. Cook Notes will typically identify the modification.

**3. Exclusion signals (Repeat? = No)**

A strong negative signal, not an automatic ban. Treat as a hard constraint only when Cook Notes show the household rejected the flavor architecture, texture profile, richness level, or technique itself. If the failure appears execution-, ingredient-, or occasion-specific, avoid repeating the failure mechanism rather than banning the entire direction. Do not name the prior dish in output unless asked — prefer neutral phrasing (e.g., "This version uses a brighter pan sauce rather than a cream-based reduction").

**4. Execution constraints (Cook Notes, Execution Rating)**

- If Cook Notes record a CCP failure on a similar dish, reinforce the corresponding CCP with heightened specificity.
- If Execution Rating is ≥1.5 points below Flavor Rating, treat the dish as technically demanding for this kitchen; prefer method simplifications that do not compromise flavor.
- Do not repeat a documented execution failure unless the request explicitly asks for it and the recipe accounts for the known risk.

**5. Recency and rotation (Last Made, Times Made)**

- If made within the last 6 weeks and Repeat? = Yes, steer silently toward complementary variety. Mention recency only if it materially changes the recommendation.
- If Times Made ≥ 3, it is a repertoire anchor. A new recipe in the same category may acknowledge this and build contrast (e.g., "leaner and more acidic counterpoint to the Wagyu dinner that anchors the beef repertoire").

---

### Vault Self-Check (Internal — Do Not Include in Output)

- [ ] Digest read before generation; used silently — not reproduced
- [ ] All five constraint types applied where relevant data exists
- [ ] Conflict hierarchy respected; current request honored over Vault signals
- [ ] Repeat? = No: strong signal, not automatic ban; failure mechanism assessed from Cook Notes
- [ ] Repeat? = With Modifications: modification applied as design constraint
- [ ] Rating confidence weighted: repeated 8.0+ outranks one-off 8.5+
- [ ] Recency used for silent steering only
- [ ] No overfitting: sparse data = directional, not prescriptive
- [ ] Vault-derived avoidances stated neutrally; only where they materially clarify direction
