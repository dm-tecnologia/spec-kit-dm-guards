---
description: "OWASP coverage, auth/authz validation, data protection checks, and secrets scan for the active feature."
handoffs:
  - label: Review Gate
    agent: speckit.dm-guard.review
    prompt: "Run code review on the active feature."
    send: true
  - label: Docs Gate
    agent: speckit.dm-guard.docs
    prompt: "Run documentation check on the active feature."
    send: true
---

## User Input

```text
$ARGUMENTS
```

## Outline

This command can run in two modes: pre-plan (security requirements review) and post-implementation (code security scan). See `skills/speckit-security/SKILL.md` for full rules.

### Step 1 — Select Mode

Load configuration: if `.specify/extensions/dm-guard/config.yml` or `config.yml` exists in the project root, read `gates` and `security` sections.

- **Pre-plan mode**: User is reviewing security requirements before `/speckit.plan`. Run `.specify/scripts/bash/check-prerequisites.sh --json --paths-only`.
- **Post-implementation mode**: User is scanning code before `speckit.promote`. Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`.

Parse JSON for `FEATURE_DIR`.

### Step 2 — Load Context

- `FEATURE_DIR/spec.md`
- `FEATURE_DIR/plan.md`
- `.specify/memory/constitution.md`

If post-implementation, also load `FEATURE_DIR/tasks.md`.

### Step 3 — OWASP Coverage Scan

Map spec requirements against OWASP Top 10 (2021): A01 through A10. For each missing or partial category, generate a checklist item in `FEATURE_DIR/checklists/security.md`.

### Step 4 — Auth/AuthZ Pattern Validation

If feature involves auth: validate roles, permissions, auth mechanism, middleware/guards, password hashing, session/token lifecycle, rate limiting, CSRF, MFA requirements.

### Step 5 — Data Protection Validation

Check PII handling, secrets storage, file upload security, API key management, data retention.

### Step 6 — Code Security Scan (Post-Implementation Only)

Scan added lines for: hardcoded secrets, shell injection, unsafe deserialization, SQL injection, disabled TLS, debug logging. Use `rg` or equivalent.

### Step 7 — Gate Decision

- All pre-implementation items pass → proceed to planning
- Post-implementation items fail → **BLOCK**
- OWASP gaps in spec → warn

### Step 8 — Write Artifacts

Write `FEATURE_DIR/checklists/security.md` (human checklist) and `FEATURE_DIR/gates/security.json` (structured status per `gate-status-schema.md`).

Report gate decision and suggest next action.

## Done When

- [ ] OWASP coverage scan completed
- [ ] Auth/authz patterns validated (if applicable)
- [ ] Code security scan completed (if post-implementation)
- [ ] `FEATURE_DIR/checklists/security.md` written
- [ ] `FEATURE_DIR/gates/security.json` written
- [ ] Gate decision reported
