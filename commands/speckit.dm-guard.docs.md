---
description: "Verify that docs, AGENTS.md, OpenAPI contracts, env examples, i18n locales, and README are synchronized with the implemented feature."
handoffs:
  - label: Promote Gate
    agent: speckit.dm-guard.promote
    prompt: "Promote the active feature after docs gate."
    send: true
scripts:
  sh: .specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
---

## User Input

```text
$ARGUMENTS
```

## Outline

This command checks that documentation targets are in sync with code changes. Targets can be configured via `config.yml` or default to project conventions. See `skills/speckit-docs/SKILL.md` for full rules.

### Step 1 — Setup

Run the prerequisites script from repo root and parse JSON for `FEATURE_DIR` and `AVAILABLE_DOCS`. Load spec, plan, tasks, and contracts if present.

### Step 2 — Validate Prior Gates

Read `FEATURE_DIR/gates/review.json`. If `blocked` or `not-run`, warn but continue (the promote gate enforces downstream). Read `FEATURE_DIR/gates/security.json` and `FEATURE_DIR/gates/verify.json` for context when present.

### Step 3 — Identify Documentation Targets

Based on `config.yml`, project structure, and `plan.md`:

| Change Type | Target |
|---|---|
| New endpoints / API changes | OpenAPI contracts, README |
| New config / env vars | `.env.example`, README |
| New modules/packages | Context files (AGENTS.md, CLAUDE.md) |
| Architecture changes | Context files, architecture docs |
| New dependencies | README prerequisites |
| Auth/authz changes | Security docs, context files |
| i18n additions | All configured locale files |

### Step 4 — Verify Each Target

Use `git diff main..HEAD` and `rg` to detect changes that require doc updates. For each target, report: updated, missing, or not-applicable.

### Step 5 — Gate Decision

- All targets updated → `passed`
- Non-critical gaps only (e.g., README not updated) → `warning`
- Critical gaps (API contract divergence, i18n locale missing, security docs stale) → `blocked`

### Step 6 — Write Artifacts

Write `FEATURE_DIR/docs-check.md` (human report with fix suggestions) and `FEATURE_DIR/gates/docs.json` (structured status per `gate-status-schema.md`).

Report gate decision and suggest next action.

## Done When

- [ ] Documentation targets identified
- [ ] Each target verified against implementation
- [ ] Gap report with fix suggestions generated
- [ ] `FEATURE_DIR/docs-check.md` written
- [ ] `FEATURE_DIR/gates/docs.json` written
- [ ] Gate decision reported
