---
description: "Post-delivery retrospective: capture learnings, architecture decisions, process gaps, and constitution proposals after a feature ships."
handoffs:
  - label: Status Dashboard
    agent: speckit.dm-guard.status
    prompt: "Show feature status dashboard."
    send: true
---

## User Input

```text
$ARGUMENTS
```

## Outline

This command guides a structured reflection after a feature is delivered. It reads feature artifacts and gate statuses to build context, then walks through reflection categories and produces a retrospective report. It does not automatically modify the constitution. See `skills/speckit-retro/SKILL.md` for full rules.

### Step 1 — Setup

Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse JSON for `FEATURE_DIR`.

If no active feature, scan recently promoted features:
```bash
ls -lt specs/*/gates/promote.json 2>/dev/null | head -5
```

### Step 2 — Load Feature History

Collect: `spec.md`, `plan.md`, `tasks.md`, `verify.md`, `review.md`, `docs-check.md`, `gates/*.json`, git log for the feature.

### Step 3 — Structured Reflection

Answer these categories:

- **What Went Well**: What worked smoothly? Which gates caught real issues?
- **What Went Wrong**: What broke? Where did spec diverge from implementation?
- **Architecture Decisions**: What trade-offs were made? Technical debt accepted?
- **Process Gaps**: Were Spec Kit phases skipped or rushed? Task breakdown accurate?
- **Constitution Impact**: Missing principles? Patterns worth codifying?

### Step 4 — Write Retrospective Report

Write `FEATURE_DIR/retro.md` with summary, metrics, reflection categories, architecture decisions, process gaps, constitution recommendations, and action items.

### Step 5 — Constitution Sync

If constitution changes are recommended, present them to the user for approval. Do not apply automatically. Tag the update with the retro reference.

### Step 6 — Report

Display key takeaways, recommended action items, and suggest `speckit-status` for dashboard view.

## Done When

- [ ] Feature history loaded from artifacts
- [ ] Structured reflection completed (all categories)
- [ ] `FEATURE_DIR/retro.md` written
- [ ] Constitution recommendations presented (if any)
- [ ] Action items captured
