# Recipe Review Protocol (v1.2, 2026-10-30)

> Companion to `recipe_governance_core.md` Section XIV. Governs the **automatic second-opinion review** every recipe and menu goes through before it is declared stable. Jin does not ask for it and does not carry text between models: the session runs the loop itself.
>
> **v1.2 (2026-10-30):** the reviewer checks that every safety control prints: one stated only in a section the printed sheet leaves out (print protocol §3.1, §3.2) is MATERIAL (§4, §5).
>
> **v1.1 (2026-10-29):** propagation check before every new round (§5); one confirmation pass after an adopted round-3 finding (§6); the reviewer brief in §4 is the single source, read by the service from this file; fallback rounds are logged and every loop is closed on record (§2, §7); the draft is the exact text that will be saved, and saving waits for the last round (§3, §7); learning is checked against the record (§8). Audit of 26 rounds, 3–29 Oct 2026: 13 of 16 material findings in rounds 2–3 were earlier fixes not carried through.

---

## 1. When this protocol applies

Automatically, without being asked, whenever a **complete recipe or menu draft** exists and before "Recipe stable" / "Menu stable" is stated (Core Section XIV). It also applies again when Jin asks for a material change to a recipe that was already reviewed.

It does not apply to a narrow cooking-method, substitution or food-science question that produces no complete recipe.

Do not ask Jin whether to run it. Do not deliver the draft and wait. Run the loop, then deliver the reviewed result.

---

## 2. Reviewer — in this order

| Order | Reviewer | Use when |
|---|---|---|
| 1 | **GPT, through the kitchen review service** (§3) | The session has the Supabase connector and the service answers. |
| 2 | **Claude critic (fallback)** | No Supabase connector, or the service returns `no_key`, `error`, or stays `pending` for more than 4 minutes. |

**Claude critic fallback.** Where the session can start a separate agent, give a fresh agent only the draft, Core (plus Meal Workflow for a menu) and the reviewer brief in §4, and treat its output as the critique. Where it cannot, write the critique as a distinct pass under the same brief before changing anything in the draft. Either way, say in the Review log that the fallback was used and why. Where the session has the Supabase connector (the service failed, not the connector), record each fallback round:

```sql
select public.kitchen_review_log_fallback(
  p_subject := 'Meal or dish name — identical in every round',
  p_round := 1,
  p_draft := $rv$ …the draft as reviewed… $rv$,
  p_critique := $rv$ …the critic's output, ending in its VERDICT line… $rv$,
  p_decision_log := null,
  p_meal_slug := 'meal-or-dish-slug',
  p_reason := 'GPT service error: no credits'
);
```

---

## 3. Calling the kitchen review service

Supabase project `umqkgxsypcgvzkfjkzpc`. Two SQL calls per round.

**Request a round:**

```sql
select public.kitchen_review_request(
  p_subject := 'Meal or dish name — identical in every round',
  p_round := 1,
  p_draft := $rv$ …the full current draft… $rv$,
  p_decision_log := null,            -- from round 2: the cumulative decision log, $rv$ … $rv$
  p_multi_component := true,         -- true for a menu, false for a single dish
  p_meal_slug := 'meal-or-dish-slug',-- the slug it will be saved under; the same in every round
  p_confirm := false                 -- true only for the confirmation pass (§6)
);
```

It returns `{"status":"requested","review_id":"…"}` plus `governance_sha` (the Git blob of the Core the reviewer was given) and `framework_loaded`. The service adds the current framework and the §4 brief itself; do not paste governance into the draft. If `framework_loaded` is false, the reviewer judged on culinary merits only: say so in the Review log.

**Collect the critique:**

```sql
select public.kitchen_review_result('<review_id>');
```

`pending` → call it again after a few seconds (a review usually takes 20–90 seconds). `done` → the reply carries `verdict` (`material_issues` | `no_material_issues` | `unclear`) and `critique`. `no_key` / `error` / `round_limit` → follow the `message`. An error that mentions credits or billing: tell Jin in one line that the OpenAI account needs topping up, then use the fallback.

**The draft** is the exact text that will be delivered and saved: for a menu, the Meal Workflow plus every dish with all its sections; for a dish, all its sections. Compose it once per round and send that text; never re-render stored data for the review, since a rendering that rounds or drops quantities makes the reviewer critique numbers the recipe does not contain. Start the draft with the diner profile line (Core, Food-safety floor). Use a dollar-quote tag that does not occur in the text.

The draft is sent to OpenAI's API for the review. Never include anything in it beyond the recipe and menu content.

---

## 4. Reviewer brief

This brief is the single source. The kitchen review service reads the text between the two markers from this file on every request; the Claude critic fallback uses the same text verbatim. Edit it here, nowhere else.

<!-- reviewer-brief:start -->
You are the independent second reviewer of a home-cooking recipe or multi-component menu written for Jin, who lives in Singapore. Household: 3 adults; 2 induction hobs, one Bosch built-in oven, Tiger rice cooker, butane torch; outdoors a 3-burner gas stove and a binchotan grill; conservative salt and sugar; moderate carbohydrates. Where the draft states another location, party or equipment, judge against what the draft states. Review the DRAFT against the governance framework supplied in the input and against sound culinary science and food-safety practice. You are read-only: critique, do not rewrite the recipe.

Report only findings that would change the result on the plate, the safety of the food, or whether the timeline can actually be executed with the stated equipment and one cook. Check arithmetic: quantities, timings, T-minus timelines at both ends of every time band, oven and burner conflicts, holding limits, carryover. Check buying: every ingredient whose result depends on a variety, cut, grade, size or brand must state what to buy, and name what not to buy where the common substitute on the shelf would fail, on the ingredient line and on the shopping-list row itself, not only in the method or a sourcing note. A missing or contradictory buying spec that would lead the shopper to buy the wrong thing is MATERIAL. Check print safety: the printed sheet leaves out the workflow's planning and check sections (failure prioritisation, heat resource allocation, oven sequencing, parallel vs sequential, final plating window, timeline verification, meal balance audit, sourcing notes) and each dish's Flavor Architecture, Nutrition Estimate and Scaling Notes. A safety control (a hold with its temperature and limit, a reheat endpoint, a cooling check with its reheat-or-discard branch, a doneness-safety check) stated only in one of those sections is MATERIAL: it must also appear in the method, a CCP, the holding chain or a timeline row.

Judge safety against the diner profile the draft states. Unless the profile declares a vulnerable diner (pregnant, immunocompromised, elderly or a young child), do not ask for a safer-endpoint alternative or a vulnerable-diner branch: a one-line statement of the trade-off is enough.

Do not raise terminology, formatting, naming or documentation points; under framework Section XIV those are not material. Where a safety control sits is not a documentation point.

If a PRIOR DECISION LOG is supplied, first check that every adopted change, and every branch, fallback or alternative it created, is carried through each place the component appears (ingredient line, shopping row, method, timeline at both ends of every band, holding chain, CCPs, storage), that any safety control it adds appears where the sheet prints it (method, CCPs, holding chain or a timeline row), and that it closes in the timeline; report any failure there before other findings. Do not re-raise a point that was declined with a stated reason unless that reason is factually wrong, and if so say exactly why.

If the input is marked CONFIRMATION PASS, check only that the changes adopted after the previous round are implemented and close in the timeline. Raise nothing else unless one of those changes caused it.

Output format, plain text: numbered findings, most important first, at most 8. Each finding: [MATERIAL] or [MINOR] · where (dish / section / step) · the problem and its mechanism · the specific change you propose, with numbers. Then a line starting "What is sound:" with one to three short points. The final line must be exactly "VERDICT: MATERIAL ISSUES" or "VERDICT: NO MATERIAL ISSUES". Use NO MATERIAL ISSUES when every remaining point is minor.
<!-- reviewer-brief:end -->

---

## 5. Reconcile every round

Sort **every** finding into exactly one of:

| Decision | Meaning |
|---|---|
| **Adopt** | The reviewer is right. Change the draft as proposed. |
| **Adopt with modification** | The problem is real; a different fix suits this household, equipment or framework better. State the fix used. |
| **Decline** | With a stated reason: a framework rule, a household constraint, a preference Jin has stated, a culinary reason, or cook evidence (a logged cook that validates the current choice is the strongest reason). |

Precedence when they conflict: Core safety rules, then Jin's explicit instructions in this conversation, then the framework, then the reviewer. A `[MATERIAL]` finding is never dropped silently. Apply the adopted changes to the draft, re-run the Core self-check on what changed, and re-verify any timeline arithmetic the change touches.

**Propagation check (before the next round).** For every adopted change that adds or alters a branch, fallback, alternative or conditional path, trace it through every place the component appears: ingredient line, shopping row, method, T-minus timeline at both ends of every band, holding chain, CCPs and storage. A safety control it adds must land where the sheet prints it: the method, a CCP, the holding chain or a timeline row. Prefer bounding the input over adding a branch (buy a 2–3 cm steak; use a pot with a flat base of at least 20 cm), per the Core complexity budget. A branch that cannot be closed in the timeline is removed or replaced by such a bound, not sent to the reviewer open. Note "propagated" in the decision-log entry once the check is done.

Keep a cumulative **decision log**: round · finding (one line) · decision · what changed or why declined.

---

## 6. Loop

1. Round 1 on the first complete draft.
2. If the verdict is `material_issues` and fewer than **3** rounds have run, complete the propagation check (§5), then request the next round with the revised draft and the cumulative decision log.
3. Stop when the verdict is `no_material_issues`, or after round 3.
4. **Confirmation pass.** If round 3 carries a `[MATERIAL]` finding that is adopted, in full or with modification, request round 4 with `p_confirm := true`, the revised draft and the cumulative decision log. The reviewer checks only the round-3 changes. A `[MATERIAL]` finding in the confirmation pass goes under **Open disagreements**; there is no round 5. A round-3 `[MATERIAL]` finding that is declined goes straight under **Open disagreements**, with no confirmation pass.
5. A material change Jin asks for after the loop has closed starts a new loop at round 1 on the revised draft, carrying the earlier decision log.

Run the whole loop inside one reply wherever the session allows. Do not show interim drafts between rounds.

---

## 7. What Jin sees, and closing the record

The final recipe or menu, then a **Review log** — not the critiques themselves:

```
Review log — reviewer: GPT (<model>) · rounds: 2 · final verdict: no material issues
Round 1
| Finding | Decision | Change |
| Duck pull temp leaves no carryover room | Adopt | Pull lowered to 128°F / 53°C |
| Add a starch | Decline | Household moderates carbs; menu already has potatoes |
Round 2
| … | … | … |
Open disagreements: none
```

Then Core Section XIV. Save only after the last round has been reconciled; never before the final round, or the confirmation pass, has been requested and answered. After the save, close the record:

```sql
select public.kitchen_review_close(
  p_subject := 'Meal or dish name — as in every round',
  p_final_decision_log := $rv$ …the full cumulative decision log, including the last round… $rv$,
  p_open_disagreements := null,      -- or the open items, one per line
  p_meal_slug := 'meal-or-dish-slug'
);
```

The closed record is the stored Review log, linked to the meal or dish by its slug; it is not added to `workflow_md` and is not printed. Then state "Recipe stable" / "Menu stable" and the save line. The full critiques stay in the `recipe_review` table; show one only if Jin asks.

---

## 8. Learning from reviews

When the same class of finding is adopted in two separate reviews, propose a row in the Governance Change Queue so the framework prevents it next time (in a session linked to Jin's computer, put it to Jin in that session). One-off findings are fixed in the recipe and not proposed.

Check the record, not memory: at the close of every loop, compare this loop's adopted findings with the closed reviews of the last 90 days —

```sql
select subject, closed_at, final_decision_log_md
from public.recipe_review
where closed_at > now() - interval '90 days'
order by closed_at desc;
```

— and name any class that has now reached the two-review bar. A finding of a class the framework already rules on is a drafting miss, not a new class: fix the draft, and add a self-check line to Core if it recurs.

---

## 9. Self-check before stating stable

- [ ] Review loop ran without being asked; reviewer named (GPT or the Claude fallback, with the reason); fallback rounds logged.
- [ ] The draft sent each round was the text that is saved and delivered; diner profile line present.
- [ ] Every finding has a decision; no `[MATERIAL]` finding dropped silently.
- [ ] Propagation check done on every adopted branch, fallback or alternative before the next round.
- [ ] Adopted changes are in the final text and the affected arithmetic was re-verified.
- [ ] Loop stopped on `no_material_issues`, or at round 3, or after the confirmation pass, with any open disagreement listed.
- [ ] Saved after the last round; record closed with `kitchen_review_close`.
- [ ] Review log shown; critiques not pasted.
