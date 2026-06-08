---
name: "speckit-security"
description: "Generate security checklist for the feature, validate OWASP coverage, scan for secrets in code, and validate auth/authz patterns against the spec."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "DM Tecnologia"
  source: "verification/speckit-security"
  tags: [speckit, security, owasp, checklist, verification]
  related_skills: [speckit-specify, speckit-review, speckit-checklist]
---

# Spec Security Gate

Security requirements validation integrated into the SpecKit flow. Generates a security checklist for the feature, validates OWASP Top 10 coverage in requirements, scans code for secrets, and validates auth/authz patterns against the spec. **Core principle:** Security is a requirements problem first, code problem second.

## When to Use

- After `speckit-specify` (security requirements review)
- Before `speckit-plan` (security architecture validation)
- After `speckit-implement` (code security scan)
- Before `speckit-promote` (final security gate)
- When feature involves: auth, payments, PII, file upload, external APIs

## Outline

### Step 1 — Setup

Select mode based on available artifacts and user intent:

- **Pre-plan mode**: run `.specify/scripts/bash/check-prerequisites.sh --json --paths-only`. Requires only `spec.md`.
- **Post-implementation mode**: run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`. Requires `tasks.md` and implementation context.

If called pre-implementation, only `spec.md` needs to exist. If called post-implementation, `tasks.md` must also exist.

### Step 2 — Load Context

- `spec.md` — Functional Requirements, User Stories, Actors/Roles
- `plan.md` — Tech stack, architecture, auth mechanism
- `.specify/memory/constitution.md` — Security principles

### Step 3 — OWASP Top 10 Coverage Scan

Map spec requirements against OWASP Top 10 (2021):

| OWASP Category | Spec Coverage | Status |
|---|---|---|
| A01: Broken Access Control | Auth roles defined? Permissions per role? | ✅/❌/⚠️ |
| A02: Cryptographic Failures | Sensitive data encrypted? TLS required? | ✅/❌/⚠️ |
| A03: Injection | Input validation required? Parameterized queries? | ✅/❌/⚠️ |
| A04: Insecure Design | Threat model? Security requirements explicit? | ✅/❌/⚠️ |
| A05: Security Misconfiguration | Secure defaults? Hardening guide? | ✅/❌/⚠️ |
| A06: Vulnerable Components | Dependency scanning? Version pinning? | ✅/❌/⚠️ |
| A07: Auth Failures | MFA? Password policy? Session management? | ✅/❌/⚠️ |
| A08: Software & Data Integrity | CI/CD integrity? Signed commits? | ✅/❌/⚠️ |
| A09: Logging & Monitoring | Audit logs? Alerting on anomalies? | ✅/❌/⚠️ |
| A10: SSRF | URL validation? Allowlist for outbound? | ✅/❌/⚠️ |

For each ❌ or ⚠️, generate a checklist item.

### Step 4 — Auth/AuthZ Pattern Validation

If the feature involves authentication or authorization:

1. **Extract auth requirements from spec.md**:
   - User roles and permissions
   - Authentication mechanism (session, JWT, OAuth2, passwordless)
   - Protected resources

2. **Validate against plan.md**:
   - Is the auth mechanism specified?
   - Are there middleware/guards for protected routes?
   - Is password hashing specified?
   - Session/token expiration defined?

3. **Common auth gaps to flag**:
   - No mention of rate limiting on login
   - Missing password complexity requirements
   - No account lockout after failed attempts
   - Token stored in localStorage (XSS risk)
   - Missing CSRF protection for cookie-based auth
   - No MFA for sensitive operations

### Step 5 — Data Protection Validation

If the feature handles sensitive data:

- **PII**: Is data minimization applied? Is there a retention policy?
- **Passwords/Secrets**: Are they hashed (bcrypt/argon2), never logged?
- **API Keys**: Are they stored in environment variables, not code?
- **File Uploads**: Type validation? Size limits? Virus scanning? Path traversal protection?

### Step 6 — Code Security Scan (Post-Implementation)

If `tasks.md` exists and implementation is done:

```bash
# Secrets in code
git diff --cached | grep "^+" | grep -iE "(password|secret|api_key|token|private_key)\s*[:=]\s*['\"][^'\"]+['\"]"

# Hardcoded URLs/IPs (SSRF risk)
git diff --cached | grep "^+" | grep -E "https?://(localhost|127\.0\.0\.1|10\.|172\.|192\.)"

# Unsafe file operations
git diff --cached | grep "^+" | grep -E "os\.Open|ioutil\.ReadFile|fs\.readFileSync" | grep -v "path\.Join\|path\.Clean\|filepath\.Clean\|path\.resolve"

# Disabled security features
git diff --cached | grep "^+" | grep -iE "insecure_skip_verify|verify=False|rejectUnauthorized.*false|DEBUG.*True"
```

### Step 7 — Generate Security Checklist

Create `FEATURE_DIR/checklists/security.md`:

```markdown
# Security Checklist: [Feature Name]

**Purpose**: Validate security requirements completeness and code security
**Created**: YYYY-MM-DD
**Feature**: [Link to spec.md]
**OWASP Version**: 2021

## OWASP Coverage

- [ ] CHK-S001 — Are access control requirements defined for all user roles? [A01, Spec §FR-X]
- [ ] CHK-S002 — Are sensitive data encrypted at rest and in transit? [A02, Gap]
- [ ] CHK-S003 — Are injection prevention requirements specified (parameterized queries, input validation)? [A03, Spec §FR-X]
- [ ] CHK-S004 — Is a threat model or security review documented? [A04, Gap]
- [ ] CHK-S005 — Are secure defaults specified (HTTP headers, CORS, CSP)? [A05, Gap]
- [ ] CHK-S006 — Are dependency versions pinned and scanning configured? [A06, Gap]
- [ ] CHK-S007 — Are authentication requirements complete (MFA, password policy, session)? [A07, Spec §FR-X]
- [ ] CHK-S008 — Are CI/CD integrity checks specified? [A08, Gap]
- [ ] CHK-S009 — Are audit logging requirements defined? [A09, Gap]
- [ ] CHK-S010 — Are SSRF protections specified for outbound requests? [A10, Gap]

## Auth/AuthZ Requirements

- [ ] CHK-S011 — Are all protected endpoints enumerated with required roles? [Completeness]
- [ ] CHK-S012 — Is the authentication mechanism specified with token/session lifecycle? [Clarity]
- [ ] CHK-S013 — Are rate limiting requirements defined for authentication endpoints? [Gap]
- [ ] CHK-S014 — Are password/token storage requirements specified (hashing algorithm)? [Clarity]

## Data Protection

- [ ] CHK-S015 — Is PII data minimization documented? [Completeness]
- [ ] CHK-S016 — Are data retention/deletion requirements specified? [Gap]
- [ ] CHK-S017 — Are file upload security requirements defined (types, sizes, scanning)? [Gap]

## Code Security

- [ ] CHK-S018 — No hardcoded secrets in code [Post-Implementation]
- [ ] CHK-S019 — No disabled TLS/certificate verification [Post-Implementation]
- [ ] CHK-S020 — All file operations use safe path construction [Post-Implementation]
```

### Step 8 — Gate Decision

- **All pre-implementation items pass**: Proceed to planning
- **Post-implementation items fail**: **BLOCK** — fix before `speckit-promote`
- **OWASP gaps in spec**: Warn — add security requirements to spec before planning

### Step 9 — Report

```text
🔒 SECURITY GATE: ⚠️ PROCEED WITH WARNINGS

OWASP Coverage: 7/10 categories addressed in spec
- Missing: A04 (Insecure Design), A06 (Vulnerable Components), A08 (Integrity)
- These may be acceptable for internal tools — review before production deploy

Code scan: ✅ Clean (no secrets, no unsafe patterns)

Checklist: FEATURE_DIR/checklists/security.md (20 items)
```

### Step 10 — Write Gate Status JSON

Create `FEATURE_DIR/gates/` if needed and write `FEATURE_DIR/gates/security.json` following `gate-status-schema.md`:

```json
{
  "schema_version": "1.0",
  "gate": "speckit-security",
  "feature": "057-whatsapp-auth",
  "status": "warning",
  "blocking": false,
  "summary": "OWASP coverage has non-blocking gaps; code scan clean",
  "findings": {
    "critical": 0,
    "high": 0,
    "medium": 3,
    "low": 0
  },
  "artifacts": ["specs/057-whatsapp-auth/checklists/security.md"],
  "evidence": [],
  "exceptions": [
    {
      "reason": "Missing A04/A06/A08 requirements are non-blocking for this internal-only feature unless production exposure changes",
      "approved_by": "user",
      "approved_at": "YYYY-MM-DDTHH:MM:SSZ"
    }
  ],
  "generated_at": "YYYY-MM-DDTHH:MM:SSZ"
}
```

## Done When

- [ ] OWASP coverage scan completed against spec
- [ ] Auth/authz patterns validated (if applicable)
- [ ] Code security scan completed (if post-implementation)
- [ ] `FEATURE_DIR/checklists/security.md` written
- [ ] `FEATURE_DIR/gates/security.json` written
- [ ] Gate decision reported
