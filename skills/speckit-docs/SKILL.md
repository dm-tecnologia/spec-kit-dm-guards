---
name: "speckit-docs"
description: "Documentation sync verification: check that docs, AGENTS.md, OpenAPI contracts, and README are updated alongside code changes."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "DM Tecnologia"
  source: "delivery/speckit-docs"
  tags: [speckit, documentation, verification, sync, contracts]
  related_skills: [speckit-plan, speckit-implement, speckit-promote]
---

# Spec Documentation Gate

Verifies that documentation stays synchronized with code. Checks AGENTS.md, OpenAPI contracts, README, and inline docs for consistency with the implemented feature. **Core principle:** Docs that don't match code are worse than no docs — they're actively misleading.

## When to Use

- After `speckit-implement` (optional gate)
- Before `speckit-promote` (recommended)
- When a feature involves: API changes, new endpoints, config changes, new dependencies
- When user's constitution requires "docs updated with code"

## Outline

### Step 1 — Setup

Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse JSON for `FEATURE_DIR`.

### Step 2 — Load Feature Context

Extract what SHOULD be documented:

- `spec.md` → Feature description, user stories
- `plan.md` → Architecture, tech decisions, new endpoints
- `tasks.md` → Tasks that reference doc updates
- `contracts/` → API contracts that should match implementation

### Step 3 — Identify Documentation Targets

Based on project structure and plan.md:

| Change Type | Doc to Check |
|---|---|
| New endpoints / API changes | `contracts/openapi.yaml`, `README.md` |
| New config / env vars | `.env.example`, `README.md` |
| New modules/packages | `AGENTS.md` (structure section) |
| Architecture changes | `AGENTS.md`, `docs/architecture.md` |
| New dependencies | `README.md` (prerequisites) |
| Auth/authz changes | `docs/security.md`, `AGENTS.md` |
| i18n additions | Both locale files (`pt-BR`, `en-US`) |

### Step 4 — Verify Doc Updates

#### AGENTS.md Check

```bash
# Check if AGENTS.md was modified in the feature branch
git diff main..HEAD -- AGENTS.md 2>/dev/null

# Check if SpecKit section points to latest plan
grep -A2 "SPECKIT START" AGENTS.md
```

Flag if AGENTS.md hasn't been updated but should have been.

#### OpenAPI Contract Check

```bash
# If contracts/ directory exists in feature
ls FEATURE_DIR/contracts/ 2>/dev/null

# Check if the actual openapi.yaml matches contracts
diff FEATURE_DIR/contracts/openapi.yaml apps/api/openapi.yaml 2>/dev/null
```

Flag if contracts diverge.

#### .env.example Check

```bash
# Find candidate env vars in changed code, then verify each appears in .env.example
git diff main..HEAD -- apps/api/ | rg '^\+.*os\.Getenv\("[A-Za-z0-9_]+"\)'
```

#### README Check

```bash
# Check if README mentions the new feature/module
grep -i "<feature-keyword>" README.md 2>/dev/null || echo "⚠️ Feature not mentioned in README"
```

#### i18n Check

```bash
# Extract i18n keys from implementation
git diff main..HEAD | rg "^\+.*t\(['\"][^'\"]+['\"]\)"

# For each key found, verify presence in every configured locale file.
```

#### Migration Docs Check (Go)

```bash
# If new migrations exist, check if documented
ls -la apps/api/migrations/*.sql 2>/dev/null
git diff main..HEAD -- apps/api/migrations/ | rg '^\+.*[0-9]{6,}_[A-Za-z0-9_]+'

# For each migration found, verify it is documented in plan.md or data-model.md.
```

### Step 5 — Build Documentation Gap Report

```text
📄 DOCUMENTATION GATE

AGENTS.md:
  ✅ Updated — SpecKit section points to specs/057/plan.md

OpenAPI Contract:
  ❌ Stale — Missing /auth/whatsapp/challenge endpoint
     Contract: contracts/whatsapp-auth.yaml (feature)
     Deployed: apps/api/openapi.yaml (missing the new endpoint)

.env.example:
  ✅ Updated — WHATSAPP_API_KEY, WHATSAPP_WEBHOOK_SECRET present

README.md:
  ⚠️ Not updated — WhatsApp auth not mentioned in Features section

i18n:
  ✅ pt-BR: 12/12 keys present
  ❌ en-US: 4/12 keys missing (whatsapp.title, whatsapp.send, whatsapp.verify, whatsapp.error)

Migrations:
  ✅ 001057_whatsapp_auth.up.sql documented in data-model.md
```

### Step 6 — Generate Fix Suggestions

For each gap, provide the exact fix:

```text
🔧 Suggested fixes:

1. OpenAPI: Copy FEATURE_DIR/contracts/whatsapp-auth.yaml → apps/api/openapi.yaml section
2. README: Add "WhatsApp Authentication" under Features
3. en-US: Add missing keys:
   "whatsapp.title": "WhatsApp",
   "whatsapp.send": "Send code",
   "whatsapp.verify": "Verify",
   "whatsapp.error": "Verification failed"
```

### Step 7 — Write Report

Save to `FEATURE_DIR/docs-check.md`:

```markdown
# Documentation Check: [Feature Name]

**Date**: YYYY-MM-DD
**Feature**: [Link to spec.md]

## Status Summary

| Target | Status | Details |
|--------|--------|---------|
| AGENTS.md | ✅ | Updated |
| OpenAPI | ❌ | Missing 1 endpoint |
| .env.example | ✅ | All vars present |
| README.md | ⚠️ | Feature not mentioned |
| i18n pt-BR | ✅ | 12/12 keys |
| i18n en-US | ❌ | 4/12 keys missing |
| Migrations | ✅ | Documented |

## Gate Decision

🚦 DOCS GATE: ❌ BLOCKED (2 critical gaps)

Critical: OpenAPI contract stale, en-US i18n missing
Non-critical: README not updated

Fix critical gaps before speckit-promote.
```

### Step 8 — Gate Decision

- **All targets ✅**: Proceed to `speckit-promote`
- **Non-critical gaps only (⚠️)**: Warn but allow proceeding
- **Critical gaps (❌): API contract, i18n, security docs**: **BLOCK**

### Step 9 — Write Gate Status JSON

Create `FEATURE_DIR/gates/` if needed and write `FEATURE_DIR/gates/docs.json` following `gate-status-schema.md`:

```json
{
  "schema_version": "1.0",
  "gate": "speckit-docs",
  "feature": "057-whatsapp-auth",
  "status": "blocked",
  "blocking": true,
  "summary": "OpenAPI contract and en-US i18n are missing critical updates",
  "findings": {
    "critical": 0,
    "high": 2,
    "medium": 1,
    "low": 0
  },
  "artifacts": ["specs/057-whatsapp-auth/docs-check.md"],
  "evidence": [
    {
      "type": "docs-sync",
      "ref": "contracts/openapi.yaml",
      "status": "failed",
      "details": "Feature contract contains endpoint missing from deployed OpenAPI file"
    }
  ],
  "exceptions": [],
  "generated_at": "YYYY-MM-DDTHH:MM:SSZ"
}
```

## Done When

- [ ] All documentation targets identified
- [ ] Each target verified against implementation
- [ ] Gap report generated with specific fixes
- [ ] `FEATURE_DIR/docs-check.md` written
- [ ] `FEATURE_DIR/gates/docs.json` written
- [ ] Gate decision reported
