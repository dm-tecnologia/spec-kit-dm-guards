---
name: "speckit-retro"
description: "Feature retrospective: capture learnings after delivery, update constitution if needed, record architectural decisions, and feed back into project memory for future features."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "DM Tecnologia"
  source: "governance/speckit-retro"
  tags: [speckit, retrospective, learning, governance, improvement]
  related_skills: [speckit-promote, speckit-constitution, speckit-status]
---

# Spec Retrospective

Post-delivery learning capture for SpecKit features. After a feature ships, this skill guides a structured reflection to capture what went well, what didn't, and what should change for the next feature. **Core principle:** Teams that don't reflect are doomed to repeat the same mistakes.

## When to Use

- After `speckit-promote` completes (optional but recommended)
- After a feature ships to production
- When user says "retro", "post-mortem", "lessons learned", "what did we learn?"
- After a particularly challenging feature (bugs, rewrites, architecture pivots)
- Periodically (every N features) to update the constitution

## Outline

### Step 1 — Setup

Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse JSON for `FEATURE_DIR`.

If no feature is active, scan for recently completed features:

```bash
ls -lt specs/*/gates/promote.json 2>/dev/null | head -5
```

### Step 2 — Load Feature History

Collect all artifacts for context:

- `spec.md` — Original requirements
- `plan.md` — Technical approach
- `tasks.md` — Implementation breakdown
- `verify.md` — Gap report (what was missed)
- `review.md` — Code quality findings
- `checklists/` — Quality checklists
- Git log for the feature branch

### Step 3 — Structured Reflection

Answer these categories:

#### What Went Well ✅

- What worked smoothly in the flow?
- Which gates caught real issues?
- Which skills/patterns saved time?

#### What Went Wrong ❌

- What broke? Where were the surprises?
- Which tasks were marked `[X]` but not actually done?
- Where did the spec diverge from implementation?
- Which reviews caught things too late?

#### Architecture Decisions 📐

- What trade-offs were made during implementation?
- Were there any architecture pivots from the original plan?
- What technical debt was intentionally accepted?

#### Process Gaps 🔧

- Were any SpecKit phases skipped or rushed?
- Did `speckit-clarify` miss important questions?
- Was the task breakdown accurate?
- Did parallel execution work as expected?

#### Constitution Impact 📜

- Did this feature reveal any missing principles?
- Should any existing principles be adjusted?
- Are there patterns worth codifying?

### Step 4 — Generate Retrospective Report

Write to `FEATURE_DIR/retro.md`:

```markdown
# Retrospective: [Feature Name]

**Date**: YYYY-MM-DD
**Feature**: [Link to spec.md]
**Delivered**: YYYY-MM-DD (commit: a1b2c3d)

## Summary

[2-3 sentence summary of what this feature was and how it went]

## Metrics

| Metric | Value |
|--------|-------|
| Total tasks | 22 |
| Tasks completed as planned | 18 |
| Tasks added during implementation | 4 |
| Verification gaps found | 3 |
| Review findings (critical) | 0 |
| Review findings (non-blocking) | 5 |
| Auto-fix cycles | 1 |
| Time from spec to promote | 4 days |

## What Went Well ✅

1. **WhatsApp integration pattern**: The passwordless challenge/verify pattern from SMS was directly reusable — saved ~40% implementation time.
2. **Verify gate caught real gaps**: 3 tasks were marked `[X]` but not actually implemented. Would have shipped broken without this gate.
3. **Security checklist**: Flagged missing rate limiting on the login endpoint before it reached production.

## What Went Wrong ❌

1. **BFF route path drift**: Tasks described `auth/passwordless/start/route.ts` but actual routes ended up at `auth-sessions/[...]/passwordless/challenges/route.ts`. Spent 2h reconciling.
2. **Monolithic component**: `AuthSessionLoginForm.tsx` grew to 1,447 lines. The plan called for modular components but implementation consolidated.
3. **i18n missed on first pass**: Only `pt-BR` locale was updated. `en-US` was caught by review.

## Architecture Decisions 📐

1. **Consolidated component vs modular**: Chose to keep `AuthSessionLoginForm` monolithic for this iteration. Extraction deferred to a dedicated refactor feature (linked: specs/060).
2. **No dedicated resend endpoint**: Backend reuses the challenge creation endpoint with client-side cooldown. Simpler, but less granular control.

## Process Gaps 🔧

1. **Clarify was skipped**: Should have run `speckit-clarify` — the BFF path question would have surfaced earlier.
2. **Plan didn't specify component granularity**: The plan said "channel selector component" but didn't specify modular vs consolidated approach.

## Constitution Recommendations

1. **Add**: "BFF route paths MUST match the canonical API structure" — prevent future path drift.
2. **Add**: "Components exceeding 500 lines MUST be flagged for extraction in plan.md" — prevent monoliths.

## Action Items

- [ ] Create spec for component extraction (specs/060-refactor-auth-form)
- [ ] Update constitution with BFF route convention
- [ ] Update speckit-clarify template to include BFF path questions
- [ ] Add i18n verification to speckit-verify

---

*Retrospective conducted per SpecKit Retro v1.0.0*
```

### Step 5 — Constitution Sync

If the retrospective identified constitution changes:

1. Present recommendations to user
2. If approved, load `speckit-constitution` to apply changes
3. Tag the constitution update with the retro reference

```text
📜 CONSTITUTION IMPACT: 2 recommendations

1. Add principle: "BFF route paths MUST match canonical API structure"
2. Add principle: "Components >500 lines MUST be flagged for extraction"

Apply these changes? (yes/no)
```

### Step 6 — Update Feature Memory

If the project uses a project memory or lessons-learned file, append key learnings:

```markdown
## Feature 057: whatsapp-auth (2026-06-05)

- WhatsApp pattern reuses SMS passwordless flow → good abstraction
- Always run speckit-clarify — BFF path questions matter
- Add i18n verification to verify gate
- Monolithic components <500 lines or flag for extraction
```

### Step 7 — Final Report

```text
📝 RETROSPECTIVE COMPLETE: 057-whatsapp-auth

Report: specs/057-whatsapp-auth/retro.md

Key Takeaways:
✅ WhatsApp pattern reuses SMS flow (saved 40% time)
❌ BFF route drift cost 2h (add clarify step next time)
📜 2 constitution amendments proposed

Next feature will benefit from:
- Mandatory clarify for API surface features
- Component size gates in plan.md
- i18n verification in verify gate
```

## Done When

- [ ] Feature history loaded from all artifacts
- [ ] Structured reflection completed (all categories)
- [ ] `FEATURE_DIR/retro.md` written
- [ ] Constitution recommendations presented (if any)
- [ ] Action items captured
- [ ] Key learnings extracted for future features
