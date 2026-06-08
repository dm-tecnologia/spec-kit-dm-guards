# Spec Kit Development Flow — DM Gates

> **Version**: 1.0.0 | **Last updated**: 2026-06-08  
> Spec Kit flow with complementary gates for verification, security, independent review, documentation, promotion, and governance.

> **Português**: See [speckit-flow.pt.md](speckit-flow.pt.md)

---

## Complete Flow

```text
FOUNDATION
  /speckit.constitution
      ↓
SPECIFICATION
  /speckit.specify
      ↓
  /speckit.clarify       (up to 5 structured questions when needed)
      ↓
  /speckit.checklist     (requirement quality and completeness)
      ↓
DESIGN
  /speckit.plan
      ↓
  /speckit.tasks
      ↓
  /speckit.taskstoissues (optional)
      ↓
ANALYSIS GATE
  /speckit.analyze       (spec.md × plan.md × tasks.md)
      ↓
IMPLEMENTATION
  /speckit.implement
      ↓
VERIFICATION GATES
  /speckit.dm-guard.verify         (spec/tasks × actual code)
      ↓
  /speckit.dm-guard.security       (security requirements + code scan)
      ↓
  /speckit.dm-guard.review         (review gate + speckit-independent-review)
      ↓
  /speckit.dm-guard.docs           (docs, contracts, and context files in sync)
      ↓
DELIVERY
  /speckit.dm-guard.promote        (validates gates, approved commit, approved push/PR)
      ↓
GOVERNANCE
  /speckit.dm-guard.retro          (lessons, ADRs, constitution proposals)
  /speckit.dm-guard.status         (feature and gate dashboard)
```

## Gates And Decisions Table

| Phase | Skill/Command | Gate | If PASS | If FAIL |
|------|---------------|------|---------|---------|
| Foundation | `/speckit.constitution` | Project principles defined | Proceed to spec | Fix constitution before starting feature |
| Specification | `/speckit.specify` | `spec.md` created without premature implementation | Run clarify/checklist | Refine spec |
| Specification | `/speckit.clarify` | Critical ambiguities resolved or explicitly deferred | Run checklist | Answer pending questions |
| Specification | `/speckit.checklist` | Requirements complete, clear, and testable | Proceed to plan | Fix spec |
| Design | `/speckit.plan` | Constitution check + design artifacts | Proceed to tasks | Fix plan |
| Design | `/speckit.tasks` | Executable `tasks.md`, ordered by user story | Run analyze | Regenerate tasks |
| Analysis | `/speckit.analyze` | Consistency spec × plan × tasks | Proceed to implement | Fix inconsistencies |
| Implementation | `/speckit.implement` | Tasks executed and marked `[X]` | Run verify | Fix implementation |
| Verify | `/speckit.dm-guard.verify` | No P1 gaps and acceptable per-story coverage | Run security | Fix gaps or approve exception |
| Security | `/speckit.dm-guard.security` | No secrets, adequate auth/authz, and blocking risks addressed | Run review | Fix security gaps |
| Review | `/speckit.dm-guard.review` | 0 critical/high blocking findings | Run docs | Fix blocking findings |
| Docs | `/speckit.dm-guard.docs` | Docs/contracts/context files in sync or not applicable | Run promote | Fix critical doc gaps |
| Promote | `/speckit.dm-guard.promote` | Required gates approved and user approved Git actions | Commit/push/PR | Fix pending gate |

## Gate Policy

- Default required gates: `verify`, `security`, `review`, `docs`, and `promote`.
- A gate may return `not-applicable` when the skill justifies by evidence that the gate does not apply to the feature.
- `promote` must not accept the mere existence of Markdown files; it must read the JSON files in `FEATURE_DIR/gates/`.
- Warnings only allow promotion when there are no blocking findings and the justification is recorded in the gate status.
- The policy may be configured per project in the future, but the default is fail-closed.

## Feature Artifacts

```text
specs/<feature>/
├── spec.md                    ← Specification (/speckit.specify)
├── plan.md                    ← Technical plan (/speckit.plan)
├── research.md                ← Technical research (plan phase 0)
├── data-model.md              ← Data model (plan phase 1)
├── quickstart.md              ← Validation guide (plan phase 1)
├── tasks.md                   ← Task list (/speckit.tasks)
├── contracts/                 ← Interface contracts
├── checklists/
│   ├── requirements.md        ← Requirements checklist (/speckit.checklist)
│   └── security.md            ← OWASP checklist (/speckit.dm-guard.security)
├── verify.md                  ← Spec/tasks × code report
├── review.md                  ← Code review report
├── docs-check.md              ← Documentation check report
├── promote.md                 ← Promotion report
├── retro.md                   ← Retrospective
└── gates/
    ├── verify.json            ← Structured verify gate status
    ├── security.json          ← Structured security gate status
    ├── review.json            ← Structured review gate status
    ├── docs.json              ← Structured docs gate status
    └── promote.json           ← Structured promote gate status
```

The schema for files under `gates/` is documented in `gate-status-schema.md`.

## Flow Properties

### Invariants

1. **Specs are the primary source of intent**: requirements and decisions must evolve in Spec Kit artifacts.
2. **Tasks are an executable plan, not proof of delivery**: `[X]` in `tasks.md` requires evidence in code.
3. **Verify is read-only by default**: reconciliations in `tasks.md` require explicit approval.
4. **Gate Markdown is not enough**: promotion decisions use structured status.
5. **Never commit, push, or open a PR without explicit permission**.
6. **No agent reviews its own code without fresh context**: `/speckit.dm-guard.review` uses `speckit-independent-review` as a local independent review.

### Parallelism

- Within a feature, tasks marked `[P]` may run in parallel when they do not touch the same files.
- Post-implementation gates run sequentially by default because each gate consumes evidence from the previous one.
- Independent review may use subagent/fresh context when the agent supports it; otherwise, it must operate in fallback mode following `speckit-independent-review`.

### Feedback Loops

- `analyze → plan/specify`: inconsistencies between spec, plan, and tasks return to the correct artifacts.
- `verify → implement`: implementation gaps return to code or to an explicit scope decision.
- `security → specify/plan/implement`: missing security requirements return to spec/plan; vulnerabilities return to code.
- `review → implement`: blocking findings return to minimal correction and re-review.
- `docs → plan/docs`: divergent contracts and docs return to documentation update or implementation correction.
- `retro → constitution`: learnings become approvable constitution/ADR proposals.

## Spec Kit Original Vs DM Gates

| Aspect | Spec Kit Original | DM Gates |
|---------|-------------------|----------|
| Specification | Yes | Uses official flow |
| Clarification | Yes | Uses official flow |
| Requirements checklist | Yes | Uses official flow |
| Design | Yes | Uses official flow |
| Tasks | Yes | Uses official flow |
| Spec × plan × tasks analysis | Yes | Uses official flow |
| Implementation | Yes | Uses official flow |
| Spec/tasks × code verification | No | `/speckit.dm-guard.verify` |
| Security gate | No | `/speckit.dm-guard.security` |
| Independent code review | Manual/ad hoc | `/speckit.dm-guard.review` + `speckit-independent-review` |
| Documentation sync | No | `/speckit.dm-guard.docs` |
| Structured commit/PR | Manual | `/speckit.dm-guard.promote` |
| Gate dashboard | No | `/speckit.dm-guard.status` |
| Retrospective/feedback | No | `/speckit.dm-guard.retro` |

## Design Principles

1. **Fail-closed**: when in doubt, block or require explicit approval.
2. **Evidence over claims**: `[X]` in `tasks.md` is not evidence; code, tests, contracts, and diffs are.
3. **Structured gates**: every gate must produce JSON status consumable by other skills.
4. **Fresh eyes**: independent review must happen with separate context when possible.
5. **Permission before action**: commit, push, PR, and task alterations require permission.
6. **Docs with code**: docs and contracts are part of the definition of done.
7. **Agent portability**: skills must not depend on a specific agent or on external skills not packaged in this repository.

## Installation

This package works as a Spec Kit extension and as standalone skills.

### Spec Kit Extension

```bash
specify extension add spec-kit-dm-guards --from https://github.com/dm-tecnologia/spec-kit-dm-guards
# or from local clone
specify extension add --dev /path/to/spec-kit-dm-guards
```

Registered commands: `/speckit.dm-guard.verify`, `/speckit.dm-guard.security`, `/speckit.dm-guard.review`, `/speckit.dm-guard.docs`, `/speckit.dm-guard.promote`, `/speckit.dm-guard.status`, `/speckit.dm-guard.retro`.

### Skills Mode

Copy or symlink the `skills/` directory into the configured agent's skills directory.

### Hooks

The extension registers optional hooks after Spec Kit phases (configure via `config.yml`):
- `after_implement` → `speckit.dm-guard.verify`
- `after_verify` → `speckit.dm-guard.security`
- `after_security` → `speckit.dm-guard.review`
- `after_review` → `speckit.dm-guard.docs`
- `after_docs` → `speckit.dm-guard.promote`

### Command Aliases (Skills Mode)

| Command | Skills-mode alias |
|---------|-------------------|
| `/speckit.dm-guard.verify` | `speckit-verify` |
| `/speckit.dm-guard.security` | `speckit-security` |
| `/speckit.dm-guard.review` | `speckit-review` |
| `/speckit.dm-guard.docs` | `speckit-docs` |
| `/speckit.dm-guard.promote` | `speckit-promote` |
| `/speckit.dm-guard.status` | `speckit-status` |
| `/speckit.dm-guard.retro` | `speckit-retro` |
