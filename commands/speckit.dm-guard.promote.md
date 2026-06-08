---
description: "Validate structured gate status JSON, prepare explicit staging plan, ask separate permission for commit, push, and PR, then record promotion evidence."
handoffs:
  - label: Retrospective
    agent: speckit.dm-guard.retro
    prompt: "Run retrospective on the delivered feature."
    send: true
  - label: Status Dashboard
    agent: speckit.dm-guard.status
    prompt: "Show feature status dashboard."
    send: true
scripts:
  sh: .specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
---

## User Input

```text
$ARGUMENTS
```

## Outline

This command validates that all required gates passed (using `gates/*.json`, not Markdown existence), then stages, commits, pushes, and opens a PR with explicit user approval at each step. See `skills/speckit-promote/SKILL.md` for full rules.

### Step 1 — Setup

Run the prerequisites script from repo root and parse JSON for `FEATURE_DIR`. Load spec, plan, tasks, all `FEATURE_DIR/gates/*.json` files, `gate-status-schema.md` if available, and config from `.specify/extensions/dm-guard/config.yml` or `config.yml` (sections: `gates`, `blocking_severity`, `promote`).

### Step 2 — Validate Gate Status

Required gates: `verify`, `security`, `review`, `docs`.

For each, read `FEATURE_DIR/gates/<gate>.json` and check:
- `schema_version` exists
- `status` is `passed`, `warning`, `blocked`, `not-applicable`, or `not-run`
- `blocking` is consistent with status

Decision:
- Missing JSON → `not-run` → block
- `blocked` → block
- `not-run` → block
- `warning` → show details, ask user to accept
- `not-applicable` → allow if justification exists
- `passed` → allow

Never decide based on Markdown file existence alone.

### Step 3 — Show Gate Summary Table

Display gate statuses, findings counts, and artifacts. If any warning or not-applicable gate exists, ask user to accept before proceeding.

### Step 4 — Build Explicit Staging Plan

Show exact file list grouped by: feature code, tests, spec artifacts, gate artifacts, documentation, configuration. Never stage unrelated files. Ask for staging approval.

### Step 5 — Generate Commit Message

Build Conventional Commit from feature context. Show message and staged diff. Ask for commit approval.

### Step 6 — Commit

After approval, stage exactly the approved files and commit. If hooks fail, stop and report. Do not bypass hooks.

### Step 7 — Push Approval

After commit, show commit hash, branch, and remote target. Ask for push approval.

### Step 8 — PR Approval

If `gh` and GitHub remote are available, detect base branch and prepare PR. Ask for PR creation approval. Skip if `gh` is unavailable.

### Step 9 — Write Promotion Artifacts

Write `FEATURE_DIR/promote.md` (human report) and `FEATURE_DIR/gates/promote.json` (structured status per `gate-status-schema.md`).

Report final status and suggest next step.

## Done When

- [ ] Required gate JSON files validated
- [ ] Warnings/exceptions accepted by user
- [ ] Staging plan approved
- [ ] Commit approved and created, or explicitly skipped
- [ ] Push approved and completed, or explicitly skipped
- [ ] PR approved and created, or explicitly skipped/unavailable
- [ ] `FEATURE_DIR/promote.md` written
- [ ] `FEATURE_DIR/gates/promote.json` written
