---
name: "speckit-promote"
description: "Final Spec Kit delivery gate. Validates structured gate status JSON, prepares an explicit staging plan, asks separate permission for commit, push, and PR, then records promotion evidence. Use only after verify, security, review, and docs gates are passed, warning-approved, or not-applicable."
compatibility: "Requires spec-kit project structure with .specify/ directory and git repository"
metadata:
  author: "DM Tecnologia"
  source: "delivery/speckit-promote"
  tags: [speckit, delivery, commit, pr, promotion, gate]
  related_skills: [speckit-verify, speckit-review, speckit-security, speckit-docs]
---

# Spec Promote Gate

Final delivery gate in the Spec Kit flow. It validates structured gate evidence, prepares a safe staging plan, asks permission before each Git action, and records promotion evidence. **Core principle:** nothing reaches the shared branch without passing gates with machine-readable evidence and explicit user approval.

## When to Use

- After `speckit-verify`, `speckit-security`, `speckit-review`, and `speckit-docs` have produced status JSON.
- Before committing, pushing, or opening a PR for a Spec Kit feature.
- When user says "promote", "commit this feature", "open PR", or "ready to merge".

## Prerequisites

- `FEATURE_DIR/gates/verify.json`
- `FEATURE_DIR/gates/security.json`
- `FEATURE_DIR/gates/review.json`
- `FEATURE_DIR/gates/docs.json`
- Git repository with a feature branch checked out.
- Explicit user approval for commit.
- Explicit user approval for push.
- Explicit user approval for PR creation.

`not-applicable` is accepted only when the gate JSON includes a clear justification.

## Outline

### Step 1 — Setup

Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse JSON for `FEATURE_DIR`.

Load:

- `FEATURE_DIR/spec.md`
- `FEATURE_DIR/plan.md`
- `FEATURE_DIR/tasks.md`
- `FEATURE_DIR/gates/*.json`
- `gate-status-schema.md` if available in the skills package

### Step 2 — Validate Gate Status JSON

Required gates by default:

- `verify`
- `security`
- `review`
- `docs`

For each required gate, read `FEATURE_DIR/gates/<gate>.json` and validate:

- `schema_version` exists.
- `gate` matches expected gate.
- `status` is one of `passed`, `warning`, `blocked`, `not-applicable`, `not-run`.
- `blocking` is consistent with status and findings.
- `artifacts` points to the human-readable report.
- `generated_at` exists.

Gate decision:

- Missing JSON → block as `not-run`.
- `blocked` → block.
- `not-run` → block.
- `warning` → allow only after showing warning details and asking the user to accept the risk.
- `not-applicable` → allow only if justification/evidence exists.
- `passed` → allow.

**Optional auto-detected gates:** After checking required gates, scan for additional gate JSON files that are not in the required list. If `FEATURE_DIR/gates/workspace.json` exists (from the `dm-workspace` extension), validate it with the same rules. When missing, skip silently — these are optional gates that only apply when their extension is installed. A `blocked` optional gate still blocks promotion.

Do not decide based on the existence of `verify.md`, `review.md`, `docs-check.md`, or `security.md` alone.

### Step 3 — Build Gate Summary

Display all gates (required + any auto-detected optional gates):

```text
| Gate | Status | Blocking | Critical | High | Medium | Low | Artifact | Required |
|------|--------|----------|----------|------|--------|-----|----------|----------|
| verify | passed | false | 0 | 0 | 1 | 0 | verify.md | yes |
| security | not-applicable | false | 0 | 0 | 0 | 0 | checklists/security.md | yes |
| review | passed | false | 0 | 0 | 0 | 2 | review.md | yes |
| docs | warning | false | 0 | 0 | 1 | 0 | docs-check.md | yes |
| workspace | passed | false | 0 | 0 | 0 | 0 | workspace/state.json | optional |
```

If any warning or not-applicable gate exists, show the summary and ask:

```text
These gates are not clean passes. Accept the documented warnings/exceptions and continue to staging? (yes/no)
```

Stop unless the user explicitly approves.

### Step 4 — Inspect Git State

Collect:

- Current branch.
- Upstream branch if configured.
- Changed files: staged, unstaged, untracked.
- Recent commits relevant to the feature.
- Remote configuration.

Do not alter the worktree during inspection.

If the branch is `main`, `master`, or a protected/shared branch, ask the user whether to create/switch to a feature branch before continuing.

### Step 5 — Build Explicit Staging Plan

Create a file list grouped by:

- Feature code.
- Tests.
- Spec artifacts.
- Gate artifacts.
- Documentation.
- Configuration.

Only include files that are part of the feature and gate evidence. Never stage unrelated work just because it exists in the worktree.

Show the exact file list:

```text
Files proposed for commit:

Feature code:
- apps/api/internal/auth/whatsapp.go

Tests:
- apps/api/internal/auth/whatsapp_test.go

Spec/gate artifacts:
- specs/057-whatsapp-auth/spec.md
- specs/057-whatsapp-auth/plan.md
- specs/057-whatsapp-auth/tasks.md
- specs/057-whatsapp-auth/verify.md
- specs/057-whatsapp-auth/review.md
- specs/057-whatsapp-auth/gates/verify.json
- specs/057-whatsapp-auth/gates/review.json
```

Ask for approval before staging:

```text
Stage exactly these files? (yes/no)
```

Stop unless approved.

### Step 6 — Generate Commit Message

Build a Conventional Commit from feature context:

```text
<type>(<scope>): <summary>

<body with implemented behavior and key layers>

Spec: specs/<feature>/spec.md
Plan: specs/<feature>/plan.md
Tasks: specs/<feature>/tasks.md

Gates:
- Verify: passed|warning|not-applicable
- Security: passed|warning|not-applicable
- Review: passed|warning|not-applicable
- Docs: passed|warning|not-applicable
```

Show the message and staged diff summary. Ask:

```text
Create commit with this message? (yes/no)
```

Commit only after explicit approval.

### Step 7 — Commit

After approval:

- Stage exactly the approved files.
- Write the commit message to a temp file.
- Commit with `git commit -F <temp-file>`.

If commit hooks fail, stop and report the failure. Do not bypass hooks.

### Step 8 — Push Approval

After commit succeeds, show:

- Commit hash.
- Branch.
- Remote target.

Ask:

```text
Push this branch to the remote? (yes/no)
```

Push only after explicit approval.

### Step 9 — PR Approval

If a GitHub remote and `gh` are available, prepare a PR title/body. Detect base branch from upstream, default branch, or ask the user. Do not assume `main`.

PR body should include:

- Feature summary.
- Spec/plan/tasks links.
- Gate summary table from JSON.
- Key changes.
- Test/validation summary.

Ask:

```text
Open this PR? (yes/no)
```

Create PR only after explicit approval.

If `gh` is unavailable or no GitHub remote exists, skip PR creation and report how far promotion completed.

### Step 10 — Write Promotion Artifacts

Save `FEATURE_DIR/promote.md`:

```markdown
# Promotion Report: [Feature Name]

**Date**: YYYY-MM-DD
**Commit**: [hash]
**Branch**: [branch]
**PR**: [url or not created]

## Gates

| Gate | Status | Findings | Artifact |
|------|--------|----------|----------|

## Files Committed

[Exact staged files]

## Approvals

- Warnings accepted: yes/no/not-needed
- Commit approved: yes
- Push approved: yes/no
- PR approved: yes/no
```

Write `FEATURE_DIR/gates/promote.json`:

```json
{
  "schema_version": "1.0",
  "gate": "speckit-promote",
  "feature": "057-whatsapp-auth",
  "status": "passed",
  "blocking": false,
  "summary": "Feature committed and PR opened",
  "findings": {
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 0
  },
  "artifacts": ["specs/057-whatsapp-auth/promote.md"],
  "evidence": [
    {
      "type": "commit",
      "ref": "a1b2c3d",
      "status": "created",
      "details": "Commit created after explicit user approval"
    }
  ],
  "exceptions": [],
  "generated_at": "YYYY-MM-DDTHH:MM:SSZ"
}
```

If commit/push/PR was intentionally skipped, use `warning` with a clear summary rather than `passed`.

### Step 11 — Final Report

Report:

- Gate status.
- Commit hash or reason commit was not created.
- Push status.
- PR URL or reason PR was not created.
- Next suggested step: `speckit-retro`.

## Done When

- [ ] Required gate JSON files validated.
- [ ] Warnings/exceptions accepted by user when present.
- [ ] Exact staging plan approved.
- [ ] Commit approved and created, or explicitly skipped.
- [ ] Push approved and completed, or explicitly skipped.
- [ ] PR approved and created, or explicitly skipped/unavailable.
- [ ] `FEATURE_DIR/promote.md` written.
- [ ] `FEATURE_DIR/gates/promote.json` written.
- [ ] Completion reported with evidence.
