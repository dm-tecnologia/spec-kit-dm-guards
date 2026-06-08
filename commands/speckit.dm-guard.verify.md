---
description: "Cross-reference spec, plan, tasks, and actual code to detect gaps, architecture drift, path changes, and unverified completed tasks before review or promote."
handoffs:
  - label: Security Gate
    agent: speckit.dm-guard.security
    prompt: "Run security gate on the active feature."
    send: true
  - label: Review Gate
    agent: speckit.dm-guard.review
    prompt: "Run code review on the active feature."
    send: true
scripts:
  sh: .specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
---

## User Input

```text
$ARGUMENTS
```

## Outline

This command is read-only by default. It does not edit `tasks.md`, `spec.md`, `plan.md`, or source code without explicit user approval. See `skills/speckit-verify/SKILL.md` for full rules and evidence types.

### Step 1 — Setup

Run the prerequisites script from repo root and parse JSON for `FEATURE_DIR` and `AVAILABLE_DOCS`.

Load configuration: if `.specify/extensions/dm-guard/config.yml` or `config.yml` exists in the project root, read `gates`, `blocking_severity`, and `verify` sections. Fall back to defaults when absent.

Load:
- `FEATURE_DIR/spec.md`
- `FEATURE_DIR/plan.md`
- `FEATURE_DIR/tasks.md`
- `.specify/memory/constitution.md` if present

### Step 2 — Build Expected Deliverables Map

For each task in `tasks.md`, extract: task ID, completion marker, story label, priority, expected file paths, expected symbols/routes/tests/docs/migrations/config.

### Step 3 — Verify Against Codebase

Search filesystem and code for evidence of each task. Evidence types: `file-exists`, `symbol-found`, `route-registered`, `test-found`, `contract-matches`, `doc-updated`, `config-updated`, `not-found`, `conflicting-evidence`.

### Step 4 — Build Gap Matrix

Classify each task:

| Status | Meaning |
|---|---|
| `verified` | Expected evidence found and matches task intent |
| `missing` | No evidence found for a completed task |
| `partial` | Some layers exist, but task intent is incomplete |
| `architecture-drift` | Equivalent functionality exists elsewhere |
| `path-changed` | Expected artifact exists at a different path |
| `unchecked-but-implemented` | Task is `[ ]`, but evidence suggests implementation exists |
| `not-started` | Task is `[ ]` and no evidence exists |

### Step 5 — Gate Decision

- All completed tasks verified and no P1 gaps → `passed`
- Minor gaps, accepted drift, or unchecked-but-implemented with no P1 blocker → `warning`
- Missing P1 task, broken user story, or >10% completed tasks missing evidence → `blocked`

### Step 6 — Write Artifacts

Write `FEATURE_DIR/verify.md` (human report) and `FEATURE_DIR/gates/verify.json` (structured status per `gate-status-schema.md`).

### Step 7 — Reconciliation Plan (No Automatic Edits)

Prepare recommended edits separately. Do not apply them unless user explicitly asks.

Report gate decision and suggest next action.

## Done When

- [ ] Gap matrix built for all tasks
- [ ] Evidence collected for completed and suspicious unchecked tasks
- [ ] No files edited unless user approved reconciliation
- [ ] `FEATURE_DIR/verify.md` written
- [ ] `FEATURE_DIR/gates/verify.json` written
- [ ] Gate decision reported with next action
