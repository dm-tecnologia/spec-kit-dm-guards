---
name: "speckit-status"
description: "Feature lifecycle dashboard: scan feature directories, report completion status per phase, show gate statuses, and identify blockers across all in-flight features."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "DM Tecnologia"
  source: "governance/speckit-status"
  tags: [speckit, status, dashboard, tracking, governance]
  related_skills: [speckit-verify, speckit-review, speckit-promote, speckit-orchestrate]
---

# Spec Status Dashboard

Feature lifecycle tracking across the SpecKit flow. Answers "what's the status of feature X?" and "what features are in flight?" without manually inspecting artifacts. Always saves both `specs/status.md` and `specs/status.html`. **Core principle:** If you can't measure it, you can't manage it.

## When to Use

- At the start of a development session ("what was I working on?")
- When coordinating multiple features
- Before sprint planning or standup
- When user asks "status", "progress", "where are we?"
- After completing any SpecKit phase

## Outline

### Step 1 — Scan Feature Directories

```bash
ls -d specs/*/ 2>/dev/null
```

If no `specs/` directory exists, report: "No features found. Start with `speckit-specify`."

### Step 2 — Detect Feature State

For each feature directory, detect which artifacts exist and their completion state:

```
specs/<feature>/
├── spec.md          ← speckit-specify completed
├── plan.md          ← speckit-plan completed
├── tasks.md         ← speckit-tasks completed
├── research.md      ← Phase 0 research
├── data-model.md    ← Phase 1 design
├── contracts/       ← Interface contracts
├── quickstart.md    ← Validation guide
├── verify.md        ← speckit-verify report
├── review.md        ← speckit-review report
├── checklists/
│   ├── requirements.md  ← speckit-specify checklist
│   └── security.md      ← speckit-security checklist
└── ...
```

### Step 3 — Compute Completion Metrics

**Phase completion detection:**

| Phase | Detection Rule |
|-------|---------------|
| Specification | `spec.md` exists |
| Clarification | `spec.md` has `## Clarifications` section |
| Planning | `plan.md` exists |
| Task Generation | `tasks.md` exists |
| Analysis | analysis report exists, if configured |
| Implementation | Tasks with `[X]` ≥ tasks with `[ ]` |
| Verification | `gates/verify.json` status |
| Security | `gates/security.json` status |
| Review | `gates/review.json` status |
| Docs | `gates/docs.json` status |
| Promotion | `gates/promote.json` status |

**Task completion from tasks.md:**

```bash
# Count completed vs total tasks
grep -c '\[x\]' specs/<feature>/tasks.md 2>/dev/null
grep -c '\[ \]' specs/<feature>/tasks.md 2>/dev/null
```

### Step 4 — Build Dashboard

#### Single Feature View

```text
┌──────────────────────────────────────────────────────┐
│  Feature: 057-whatsapp-auth                           │
│  Status:   IN REVIEW (90% complete)                   │
├──────────────────────────────────────────────────────┤
│                                                       │
│  📝 Specify    ✅  spec.md (2026-06-01)               │
│  🔍 Clarify    ✅  3 questions resolved               │
│  📐 Plan        ✅  plan.md + data-model.md           │
│  📋 Tasks       ✅  22 tasks (P1: 12, P2: 8, P3: 2)  │
│  🔬 Analyze     ✅  No critical issues                │
│  ⚙️ Implement   ✅  22/22 tasks [X]                   │
│  🔎 Verify      ✅  22/22 verified                    │
│  🔒 Security    ⚠️  18/20 checklist items             │
│  👁️ Review      🔄  In progress                       │
│  🚀 Promote     ⏳  Blocked by review                  │
│  📝 Retro       ⏳  Pending                            │
│                                                       │
│  Blockers: None                                       │
│  Next:     Complete review → promote                  │
└──────────────────────────────────────────────────────┘
```

#### Multi-Feature Dashboard

```text
┌──────────┬────────────────────┬──────────┬──────────┬──────────┐
│ Feature  │ Name               │ Progress │ Phase    │ Blocked  │
├──────────┼────────────────────┼──────────┼──────────┼──────────┤
│ 057      │ whatsapp-auth      │ 90% ██▌  │ Review   │ No       │
│ 058      │ mfa-recovery       │ 45% █▌   │ Implement│ No       │
│ 059      │ audit-logging      │ 15% ▌    │ Plan     │ No       │
│ 055      │ password-reset     │ 100% ✅  │ Delivered│ --       │
└──────────┴────────────────────┴──────────┴──────────┴──────────┘
```

### Step 5 — Identify Blockers

Scan for blocking conditions:

1. **Missing prerequisite phase**: e.g., `speckit-plan` not run but `tasks.md` exists
2. **Failed gate**: any required `gates/<gate>.json` has `status=blocked` or `blocking=true`
3. **Missing required gate**: expected gate JSON is absent, treated as `not-run`
4. **Outdated artifacts**: `spec.md` modified after `plan.md` (drift)
5. **Cross-feature conflict**: Two features touching same files (see `speckit-orchestrate`)

### Step 6 — Generate Recommendations

Based on current state, suggest next action:

```text
💡 Recommendations:
1. 057-whatsapp-auth: Complete speckit-review → speckit-promote
2. 058-mfa-recovery: Implement remaining 5 tasks (T040-T044)
3. 059-audit-logging: Run speckit-tasks to break down plan
```

### Step 7 — Output Format

Always write two files:

**`specs/status.md`** — the dashboard in Markdown format.

**`specs/status.html`** — a self-contained HTML page following the template at `status-template.html`. Fill the placeholders documented in the template: `__GENERATED_AT__`, `__FEATURE_COUNT__`, `__SUMMARY_CARDS__`, `__ATTENTION_SECTION__`, `__RECOMMENDATIONS__`, and `__FEATURES_JSON_PLACEHOLDER__`.

Display the dashboard inline after saving.

## Done When

- [ ] All feature directories scanned
- [ ] Phase completion detected for each feature
- [ ] Task completion % computed
- [ ] Blockers identified
- [ ] Recommendations provided
- [ ] `specs/status.md` written
- [ ] `specs/status.html` written
- [ ] Dashboard displayed to user
