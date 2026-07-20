# Cowork Drain Procedure — Governance Change Queue

> Run on a schedule (suggested: daily, or weekly on Sunday morning before menu planning).
> Cowork is the **only** writer to this repo. Claude and GPT propose via Notion; they never edit files here.

---

## Inputs

- **Queue:** Notion database `Governance Change Queue`
  `https://app.notion.com/p/466bfaf3d8ed43dfa0e6a4c28caae2ef`
- **Hub page (changelog target):** `https://app.notion.com/p/3a1e0391ce6381dfb6dfc3b0c9e2e612`
- **Repo:** this repository, local clone

---

## Procedure

### 1. Fetch approved rows

Query the queue for `Status = "Approved"`. Sort ascending by `Date Proposed` so sequential edits to the same section land in order.

If there are no approved rows, stop. Report "queue empty."

### 2. Sync the repo

```bash
git pull --ff-only
```

If the pull fails or the working tree is dirty, **stop and report**. Do not force, stash, or merge.

### 3. Apply each row

For each row, in order:

| Operation | Action |
|---|---|
| `replace` | Find **Anchor Text** in the target file. It must appear **exactly once**. Replace it with **New Text**. |
| `add` | If **Anchor Text** is present, insert **New Text** immediately after it. If blank, append **New Text** to the end of the named **Section**. |
| `delete` | Find **Anchor Text** and remove it. Must appear exactly once. |
| `version-bump` | Update the version in the file's H1 header line only. |

**Anchor matching is strict.** If **Anchor Text** does not match exactly, or matches more than once:

- Do **not** guess, fuzzy-match, or partially apply
- Leave the row at `Approved`
- Write the error into the **Commit** field (e.g. `ANCHOR FAIL: 0 matches in recipe_governance_core.md`)
- Continue to the next row
- Report all failures at the end

A failed anchor usually means the proposal was written against a stale version of the file.

### 4. Bump versions

Where **Version After** is set, update:

- The version in the target file's H1 header
- The matching `version` field in `manifest.json`
- `framework_version` and `updated` in `manifest.json` if a core file changed

### 5. Commit and push

```bash
git add -A
git commit -m "governance: <short summary> (<n> change(s) from queue)"
git push
```

### 6. Verify

Re-fetch each changed file from its raw URL and confirm **New Text** is present. A successful push that doesn't produce the expected content is a silent failure — report it if it occurs.

```bash
curl -s https://raw.githubusercontent.com/<REPO>/main/<file> | grep -F "<distinctive phrase from New Text>"
```

### 7. Write back to Notion

For each successfully applied row:

- `Status` → `Applied`
- `Date Applied` → today
- `Commit` → the commit SHA

### 8. Update the changelog

Append to the hub page's Changelog section: the version, date, and a one-line entry per applied change including its **Rationale** trigger.

---

## Reporting

End the run with a short summary:

```
Applied: 3 | Failed: 1 | Skipped: 0
Commit: a1b2c3d
FAILED — "Cue taxonomy strengthening": ANCHOR FAIL, 0 matches in recipe_governance_core.md
```

---

## Guardrails

- Never apply a row that is not `Approved` — `Proposed` rows are awaiting Jin's review
- Never edit a governance file outside this procedure
- Never delete a file
- Never rewrite history (no `--force`, no rebase of pushed commits)
- If two approved rows target the same **Anchor Text**, apply oldest first; the second will likely fail its anchor, which is the correct signal that it needs rewriting against the updated file
