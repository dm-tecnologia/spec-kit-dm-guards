---
description: "Static security scan, safe quality checks, and independent code review on feature diff. Uses the local speckit-independent-review skill without external dependencies."
handoffs:
  - label: Docs Gate
    agent: speckit.dm-guard.docs
    prompt: "Run documentation check on the active feature."
    send: true
  - label: Promote Gate
    agent: speckit.dm-guard.promote
    prompt: "Promote the active feature."
    send: true
scripts:
  sh: .specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
---

## User Input

```text
$ARGUMENTS
```

## Outline

This command runs security scans and quality checks, then invokes the local `speckit-independent-review` skill for a fresh-context review. It does not depend on external review skills. Never use `git stash` or mutate the worktree. See `skills/speckit-review/SKILL.md` for full rules.

### Step 1 — Setup

Run the prerequisites script from repo root and parse JSON for `FEATURE_DIR` and `AVAILABLE_DOCS`. Load spec, plan, tasks, prior gate statuses from `FEATURE_DIR/gates/`, `gate-status-schema.md` if available, and config from `.specify/extensions/dm-guard/config.yml` or `config.yml` (sections: `gates`, `blocking_severity`, `review`).

### Step 2 — Validate Prior Gates

- `gates/verify.json` should be `passed` or `warning` with documented justification.
- `gates/security.json` if present and `blocked`, stop and report.

### Step 3 — Select Diff Source (No Worktree Mutation)

Never use `git stash`, `git reset`, or `git checkout`. Select first available:
1. User-provided commit range or base ref.
2. Staged: `git diff --cached`.
3. Unstaged: `git diff`.
4. Branch diff: `git diff $(git merge-base HEAD origin/main)..HEAD`.
5. Last commit: `git diff HEAD~1 HEAD`.

### Step 4 — Static Security Scan

Scan added lines for hardcoded secrets, injection patterns, unsafe deserialization, disabled TLS, debug logging. Classify as `critical`, `high`, `medium`, or `low`.

### Step 5 — Safe Quality Checks

Infer project commands from `plan.md` and package files. Run tests, lint, typecheck if available. Do not stash. Record `baseline_unavailable` if no safe baseline exists.

### Step 6 — Independent Review

Invoke the local `speckit-independent-review` skill. Use subagent/fresh-context mode when available; otherwise fallback to manual review following `skills/speckit-independent-review/SKILL.md`. The reviewer must not edit files.

### Step 7 — Gate Decision

- Any `critical` finding → `blocked`
- Any `high` finding → `blocked` by default
- New test/lint/typecheck regression → `blocked`
- Only `medium`/`low` findings → `warning`
- No findings/regressions → `passed`

### Step 8 — Write Artifacts

Write `FEATURE_DIR/review.md` (human report) and `FEATURE_DIR/gates/review.json` (structured status per `gate-status-schema.md`).

Report gate decision and suggest next action.

## Done When

- [ ] Prior gate statuses checked
- [ ] Diff source selected without worktree mutation
- [ ] Static security scan completed
- [ ] Quality checks run or explicitly marked unavailable
- [ ] Independent review completed
- [ ] `FEATURE_DIR/review.md` written
- [ ] `FEATURE_DIR/gates/review.json` written
- [ ] Gate decision reported
