---
name: "speckit-verify"
description: "Read-only Spec Kit verification gate. Cross-references spec, plan, tasks, and actual code to detect implementation gaps, architecture drift, path changes, and unverified completed tasks before review or promote."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "DM Tecnologia"
  source: "verification/speckit-verify"
  tags: [speckit, verification, sdd, audit, quality]
  related_skills: [speckit-implement, speckit-review, speckit-promote]
---

# Spec Verification Gate

Read-only verification gate between `/speckit.implement` and `speckit-review`. It cross-references `spec.md`, `plan.md`, `tasks.md`, and the codebase to determine whether completed tasks have real evidence. **Core principle:** `[X]` in `tasks.md` is a claim; code, tests, contracts, and docs are evidence.

## When to Use

- After `/speckit.implement` completes.
- Before `speckit-review` and `speckit-promote`.
- When reviewing a PR that references a Spec Kit feature.
- When user asks "is this feature really done?" or "verify spec against code".

## Prerequisites

- Feature directory with `spec.md`, `plan.md`, and `tasks.md`.
- Git repository or readable workspace.
- No requirement to stage or commit changes.

## Operating Rules

- Read-only by default. Do not edit `tasks.md`, `spec.md`, `plan.md`, or code.
- Do not uncheck tasks automatically.
- Produce a reconciliation plan when task status appears wrong.
- Ask explicit user approval before making any follow-up edits.
- Treat tasks as stable anchors: never delete or renumber task IDs.

## Outline

### Step 1 — Setup

Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse JSON for `FEATURE_DIR` and `AVAILABLE_DOCS`. All paths must be absolute.

If the script fails or JSON parsing fails, stop and instruct the user to run `/speckit.tasks` or verify the active feature branch.

### Step 2 — Load Feature Context

Read:

```text
FEATURE_DIR/spec.md
FEATURE_DIR/plan.md
FEATURE_DIR/tasks.md
```

If present, also read:

- `.specify/memory/constitution.md`
- `FEATURE_DIR/contracts/`
- `FEATURE_DIR/data-model.md`
- `FEATURE_DIR/quickstart.md`

### Step 3 — Build Expected Deliverables Map

For each task in `tasks.md`, extract:

- Task ID.
- Completion marker: `[X]` or `[ ]`.
- User story label, e.g. `[US1]`.
- Priority inferred from user story and spec.
- Expected file paths.
- Expected symbols, routes, tests, docs, migrations, i18n keys, contracts, or config.

Task patterns:

| Task Pattern | Evidence To Look For |
|---|---|
| `Create X in path/to/file.ts` | File exists at expected path |
| `Implement X handler/service` | Function/class/struct exists and is wired |
| `Add X route/endpoint` | Route file or router registration exists |
| `Write tests for X` | Relevant test file and assertions exist |
| `Update docs/contracts` | Doc/contract changed and matches behavior |
| `Add i18n keys` | Keys exist in configured locales |
| `Add migration` | Migration exists and maps to data model |

### Step 4 — Verify Against Codebase

Use file search and content search to find evidence. Prefer exact paths from tasks first; then search likely project directories from `plan.md`.

Evidence types:

- `file-exists`
- `symbol-found`
- `route-registered`
- `test-found`
- `contract-matches`
- `doc-updated`
- `config-updated`
- `not-found`
- `conflicting-evidence`

### Step 5 — Build Gap Matrix

Classify each task:

| Status | Meaning | Gate Impact |
|---|---|---|
| `verified` | Expected evidence found and matches task intent. | Pass |
| `missing` | No evidence found for a completed task. | Blocks if P1/high impact |
| `partial` | Some layers exist, but task intent is incomplete. | Warning or block by impact |
| `architecture-drift` | Equivalent functionality exists elsewhere but task target differs. | Warning unless story behavior is broken |
| `path-changed` | Expected artifact exists at a different path. | Warning |
| `unchecked-but-implemented` | Task is `[ ]`, but evidence suggests implementation exists. | Warning and reconciliation candidate |
| `not-started` | Task is `[ ]` and no evidence exists. | Informational unless required for completed story |

Example matrix:

| Task | Marker | Story | Expected | Status | Evidence |
|---|---|---|---|---|---|
| T017 | `[X]` | US1 | `auth-client.ts` supports WhatsApp | verified | Found `whatsapp` branch in `auth-client.ts` |
| T024 | `[X]` | US1 | `channel-selector.tsx` | architecture-drift | Functionality consolidated in `AuthSessionLoginForm.tsx` |
| T040 | `[ ]` | US2 | resend endpoint | unchecked-but-implemented | Route found in `apps/api/auth/resend.go` |

### Step 6 — Detect Architecture Drift

Drift is not automatically failure. Classify carefully:

- `missing`: expected behavior does not appear anywhere.
- `architecture-drift`: behavior appears implemented, but structure differs from tasks/plan.
- `path-changed`: artifact exists at a different path with equivalent role.
- `scope-change`: implementation intentionally differs from plan and should be reflected in spec/plan/tasks after approval.

### Step 7 — Reconciliation Plan (No Automatic Edits)

Prepare recommended edits separately from the verification decision.

Examples:

```markdown
## Reconciliation Plan

1. T024 is marked `[X]`, but target file does not exist. Functionality appears consolidated in `AuthSessionLoginForm.tsx`.
   Recommendation: choose one:
   - Update tasks.md note to document accepted architecture drift.
   - Implement the planned separate component.

2. T040 is `[ ]`, but route exists and tests pass.
   Recommendation: mark T040 `[X]` after user confirms evidence.
```

Do not apply this plan unless the user explicitly asks.

### Step 8 — Gate Decision

Default thresholds:

- All completed tasks verified and no P1 gaps → `passed`.
- Minor gaps, accepted drift, or unchecked-but-implemented findings with no P1 blocker → `warning`.
- Any missing P1 task, broken user story, or more than 10% completed tasks missing evidence → `blocked`.

Severity mapping:

- `critical`: P1 user story cannot work or core security/contract evidence is absent.
- `high`: completed task lacks evidence and blocks a user-visible scenario.
- `medium`: drift, partial implementation, missing tests/docs with workaround.
- `low`: path mismatch, minor task bookkeeping issue.

### Step 9 — Write Verification Report

Save to `FEATURE_DIR/verify.md`:

```markdown
# Verification Report: [Feature Name]

**Date**: YYYY-MM-DD
**Feature**: [link to spec.md]

## Verdict

🚦 **VERIFY GATE: passed|warning|blocked**

## Summary

| Metric | Value |
|--------|-------|
| Total tasks | N |
| Completed `[X]` | A |
| Verified | B |
| Missing | C |
| Partial | D |
| Drift | E |
| Unchecked but implemented | F |

## Gap Matrix

| Task | Marker | Story | Expected | Status | Evidence |
|------|--------|-------|----------|--------|----------|

## Coverage By Story

| User Story | Tasks | Verified | Missing | Coverage % |
|------------|-------|----------|---------|------------|

## Reconciliation Plan

[Recommended edits, not applied automatically.]

## Gate Decision

[Explain pass/warning/block rationale.]
```

### Step 10 — Write Gate Status JSON

Create `FEATURE_DIR/gates/` if needed and write `FEATURE_DIR/gates/verify.json` following `gate-status-schema.md`:

```json
{
  "schema_version": "1.0",
  "gate": "speckit-verify",
  "feature": "057-whatsapp-auth",
  "status": "warning",
  "blocking": false,
  "summary": "21/22 completed tasks verified; 1 architecture drift documented",
  "findings": {
    "critical": 0,
    "high": 0,
    "medium": 1,
    "low": 0
  },
  "artifacts": ["specs/057-whatsapp-auth/verify.md"],
  "evidence": [
    {
      "type": "task-verification",
      "ref": "T024",
      "status": "architecture-drift",
      "details": "Functionality found in AuthSessionLoginForm.tsx instead of channel-selector.tsx"
    }
  ],
  "exceptions": [],
  "generated_at": "YYYY-MM-DDTHH:MM:SSZ"
}
```

### Step 11 — Report Next Action

- `passed` → suggest `speckit-security`.
- `warning` → show warnings and ask whether to reconcile, accept drift, or proceed.
- `blocked` → list blockers and suggest returning to implementation.

## Done When

- [ ] Feature context loaded.
- [ ] Deliverables map built for all tasks.
- [ ] Evidence collected for completed and suspicious unchecked tasks.
- [ ] Gap matrix and story coverage generated.
- [ ] No files edited unless user explicitly approved a follow-up reconciliation.
- [ ] `FEATURE_DIR/verify.md` written.
- [ ] `FEATURE_DIR/gates/verify.json` written.
- [ ] Gate decision reported with next action.
