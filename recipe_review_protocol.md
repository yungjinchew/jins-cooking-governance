# Recipe Review Protocol (v1.0, 2026-10-02)

> Companion to `recipe_governance_core.md` Section XIV. Governs the **automatic second-opinion review** every recipe and menu goes through before it is declared stable. Jin does not ask for it and does not carry text between models: the session runs the loop itself.

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

**Claude critic fallback.** Where the session can start a separate agent, give a fresh agent only the draft, Core (plus Meal Workflow for a menu) and the reviewer brief in §4, and treat its output as the critique. Where it cannot, write the critique as a distinct pass under the same brief before changing anything in the draft. Either way, say in the Review log that the fallback was used and why.

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
  p_multi_component := true          -- true for a menu, false for a single dish
);
```

It returns `{"status":"requested","review_id":"…"}`. The service adds the current framework itself; do not paste governance into the draft.

**Collect the critique:**

```sql
select public.kitchen_review_result('<review_id>');
```

`pending` → call it again after a few seconds (a review usually takes 20–90 seconds). `done` → the reply carries `verdict` (`material_issues` | `no_material_issues` | `unclear`) and `critique`. `no_key` / `error` / `round_limit` → follow the `message`.

**The draft** is the full current text exactly as it would be delivered: for a menu, the Meal Workflow plus every dish with all its sections; for a dish, all its sections. Use a dollar-quote tag that does not occur in the text.

The draft is sent to OpenAI's API for the review. Never include anything in it beyond the recipe and menu content.

---

## 4. Reviewer brief (used by the service; reuse it verbatim for the fallback)

Independent second reviewer of a home-cooking recipe or menu for Jin's household (3 adults; 2 induction hobs, one Bosch oven, Tiger rice cooker, butane torch; outdoor 3-burner gas and binchotan; conservative salt and sugar; moderate carbohydrates). Review against the framework and against sound culinary science and food safety. Read-only: critique, do not rewrite. Report only findings that would change the result on the plate, the safety of the food, or whether the timeline can be executed with the stated equipment and one cook. Check arithmetic — quantities, timings, T-minus timelines at both ends of every band, oven and burner conflicts, holding limits, carryover. No terminology, formatting, naming or documentation points. Do not re-raise a point declined with a stated reason unless the reason is factually wrong. At most 8 findings, each tagged `[MATERIAL]` or `[MINOR]`, with location, mechanism and a specific numeric change. Final line: `VERDICT: MATERIAL ISSUES` or `VERDICT: NO MATERIAL ISSUES`.

---

## 5. Reconcile every round

Sort **every** finding into exactly one of:

| Decision | Meaning |
|---|---|
| **Adopt** | The reviewer is right. Change the draft as proposed. |
| **Adopt with modification** | The problem is real; a different fix suits this household, equipment or framework better. State the fix used. |
| **Decline** | With a stated reason: a framework rule, a household constraint, a preference Jin has stated, or a culinary reason. |

Precedence when they conflict: Core safety rules, then Jin's explicit instructions in this conversation, then the framework, then the reviewer. A `[MATERIAL]` finding is never dropped silently. Apply the adopted changes to the draft, re-run the Core self-check on what changed, and re-verify any timeline arithmetic the change touches.

Keep a cumulative **decision log**: round · finding (one line) · decision · what changed or why declined.

---

## 6. Loop

1. Round 1 on the first complete draft.
2. If the verdict is `material_issues` and fewer than **3** rounds have run, request the next round with the revised draft and the cumulative decision log.
3. Stop when the verdict is `no_material_issues`, or after round 3.
4. If round 3 still carries a `[MATERIAL]` finding that was declined, list it under **Open disagreements** for Jin to decide. Do not loop further.
5. A material change Jin asks for after the loop has closed starts a new loop at round 1 on the revised draft, carrying the earlier decision log.

Run the whole loop inside one reply wherever the session allows. Do not show interim drafts between rounds.

---

## 7. What Jin sees

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

Then Core Section XIV: save, state "Recipe stable" / "Menu stable", and the save line. The full critiques stay in the `recipe_review` table; show one only if Jin asks.

---

## 8. Learning from reviews

When the same class of finding is adopted in two separate reviews, propose a row in the Governance Change Queue so the framework prevents it next time. One-off findings are fixed in the recipe and not proposed.

---

## 9. Self-check before stating stable

- [ ] Review loop ran without being asked; reviewer named (GPT or the Claude fallback, with the reason).
- [ ] Every finding has a decision; no `[MATERIAL]` finding dropped silently.
- [ ] Adopted changes are in the final text and the affected arithmetic was re-verified.
- [ ] Review log shown; critiques not pasted.
- [ ] Loop stopped on `no_material_issues` or at round 3, with any open disagreement listed.
