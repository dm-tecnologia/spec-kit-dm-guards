---
description: "Feature lifecycle dashboard: scan feature directories, report gate status from structured JSON, identify blockers, and suggest next actions."
---

## User Input

```text
$ARGUMENTS
```

## Outline

This command scans all feature directories under `specs/` and reports their status using structured gate JSON and artifact presence. Always saves both `specs/status.md` (Markdown) and `specs/status.html` (self-contained HTML dashboard).

### Step 1 — Scan Feature Directories

```bash
ls -d specs/*/ 2>/dev/null
```

If no `specs/` directory exists, report: "No features found."

### Step 2 — Detect Feature State

For each feature, detect phase completion:

| Phase | Detection Rule |
|---|---|
| Specification | `spec.md` exists |
| Clarification | `spec.md` has `## Clarifications` section |
| Planning | `plan.md` exists |
| Task Generation | `tasks.md` exists |
| Analysis | Analysis report exists if configured |
| Implementation | Tasks `[X]` ≥ tasks `[ ]` |
| Verification | `gates/verify.json` status |
| Security | `gates/security.json` status |
| Review | `gates/review.json` status |
| Docs | `gates/docs.json` status |
| Promotion | `gates/promote.json` status |

Gate statuses are read from JSON, not inferred from Markdown file existence.

### Step 3 — Compute Task Completion

Count `[x]` and `[ ]` checkboxes in `tasks.md` for each feature.

### Step 4 — Build Dashboard

Display single-feature or multi-feature view with phase completion, gate status, task progress, and blockers.

### Step 5 — Identify Blockers

- Missing prerequisite phase.
- Any required gate JSON with `status=blocked` or `blocking=true`.
- Missing required gate JSON (treated as `not-run`).
- `spec.md` modified after `plan.md` (drift).

### Step 6 — Generate Recommendations

Suggest next action for each feature based on current state.

### Step 7 — Save Artifacts

Always write two files:

**`specs/status.md`** — the dashboard in Markdown format (the same content displayed inline).

**`specs/status.html`** — a self-contained HTML page following the template at `skills/speckit-status/status-template.html`.

Placeholders to fill:

| Placeholder | Content |
|---|---|
| `__GENERATED_AT__` | ISO 8601 timestamp |
| `__FEATURE_COUNT__` | Number of features scanned |
| `__SUMMARY_CARDS__` | Summary cards HTML (complete, in progress, blocked counts) |
| `__ATTENTION_SECTION__` | "Requires Attention" section HTML or empty |
| `__RECOMMENDATIONS__` | Recommendations list items HTML |
| `__FEATURES_JSON_PLACEHOLDER__` | JS array of feature objects with shape: `{num, key, spec, clarif, plan, total, done, gates: {gate: {status, blocking, summary, critical, high, medium, low}}}` |

Display the dashboard inline after saving.

## Done When

- [ ] All feature directories scanned
- [ ] Phase completion detected per feature
- [ ] Gate statuses read from JSON
- [ ] Task completion computed
- [ ] Blockers identified
- [ ] `specs/status.md` written
- [ ] `specs/status.html` written
- [ ] Dashboard and recommendations displayed
