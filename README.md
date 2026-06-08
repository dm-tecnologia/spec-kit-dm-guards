# Spec Kit - DM Guards

Self-contained extension for the Spec Kit development flow (Specification-Driven Development). Complements the official GitHub Spec Kit with verification, independent review, security, documentation, promotion, status, and retrospective gates.

**Version**: 1.0.0

**Author**: DM Tecnologia

**Compatibility**: Requires a Spec Kit initialized project (`.specify/`) and an agent supported by `specify integration`

> **Português**: See [README.pt.md](README.pt.md)

---

## Structure

```text
spec-kit-dm-guards/
├── README.md
├── README.pt.md
├── LICENSE
├── extension.yml
├── config-template.yml
├── gate-status-schema.md
├── speckit-flow.md
├── speckit-flow.pt.md
├── commands/
│   ├── speckit.dm-guard.verify.md
│   ├── speckit.dm-guard.security.md
│   ├── speckit.dm-guard.review.md
│   ├── speckit.dm-guard.docs.md
│   ├── speckit.dm-guard.promote.md
│   ├── speckit.dm-guard.status.md
│   └── speckit.dm-guard.retro.md
└── skills/
    ├── speckit-docs/
    ├── speckit-independent-review/
    ├── speckit-promote/
    ├── speckit-retro/
    ├── speckit-review/
    ├── speckit-security/
    ├── speckit-status/
    └── speckit-verify/
```

> The `skills/` directory contains the same commands in skills-mode format for integrations that use skills instead of slash commands. Both formats are kept in sync.

## Complete Flow

```text
FOUNDATION
  /speckit.constitution
      ↓
SPECIFICATION
  /speckit.specify → /speckit.clarify → /speckit.checklist
      ↓
DESIGN
  /speckit.plan → /speckit.tasks → /speckit.taskstoissues (optional)
      ↓
ANALYSIS
  /speckit.analyze
      ↓
IMPLEMENTATION
  /speckit.implement
      ↓
VERIFICATION GATES
  /speckit.dm-guard.verify → /speckit.dm-guard.security → /speckit.dm-guard.review → /speckit.dm-guard.docs
      ↓
DELIVERY
  /speckit.dm-guard.promote
      ↓
GOVERNANCE
  /speckit.dm-guard.retro / /speckit.dm-guard.status
```

## Commands

### Critical

| Command | Purpose | Why needed |
|---------|---------|------------|
| `/speckit.dm-guard.verify` | Cross-reference spec/tasks × actual code. Detects completed tasks without evidence, architecture drift, and per-story coverage gaps. | `[X]` in `tasks.md` does not prove implementation exists. |
| `/speckit.dm-guard.review` | Code review gate with security scan, baseline-aware checks, and local independent review. | No agent should review its own code without fresh context. |
| `/speckit.dm-guard.promote` | Final gate: validates prior gate statuses, generates conventional commit, asks permission, and opens a PR with linked artifacts. | Closes traceability from spec, through code and gates, to PR. |

### High

| Command | Purpose |
|---------|---------|
| `/speckit.dm-guard.security` | Generates OWASP checklist, validates auth/authz, sensitive data, and secrets. |
| `/speckit.dm-guard.status` | Lifecycle dashboard per feature, phase, gate, and blocker. |
| `/speckit.dm-guard.retro` | Captures lessons learned, architecture decisions, and constitution proposals. |

### Quality

| Command | Purpose |
|---------|---------|
| `/speckit.dm-guard.docs` | Verifies docs, contracts, context files, env examples, and i18n are in sync. |

### Internal

`speckit-independent-review` is an internal skill invoked by `/speckit.dm-guard.review`. It receives a diff, minimal context, and scan results, and returns a structured verdict without external skill dependencies. It has no standalone command.

## Usage

### Install As Spec Kit Extension

```bash
# From local clone
specify extension add --dev /path/to/spec-kit-dm-guards

# From GitHub
specify extension add spec-kit-dm-guards --from https://github.com/dm-tecnologia/spec-kit-dm-guards
```

This registers the commands in the configured agent.

### Typical Feature Flow

```bash
# 1. Specification
/speckit.specify "Add WhatsApp authentication"
/speckit.clarify
/speckit.checklist

# 2. Design
/speckit.plan
/speckit.tasks

# 3. Analysis gate
/speckit.analyze

# 4. Implementation
/speckit.implement

# 5. DM Guard gates
/speckit.dm-guard.verify
/speckit.dm-guard.security
/speckit.dm-guard.review
/speckit.dm-guard.docs

# 6. Delivery
/speckit.dm-guard.promote

# 7. Governance
/speckit.dm-guard.retro
/speckit.dm-guard.status
```

## Gate Artifacts

Each gate produces a human-readable Markdown report and a structured JSON status following `gate-status-schema.md`.

```text
specs/<feature>/
├── verify.md
├── review.md
├── docs-check.md
├── promote.md
├── checklists/
│   └── security.md
└── gates/
    ├── verify.json
    ├── security.json
    ├── review.json
    ├── docs.json
    └── promote.json
```

`/speckit.dm-guard.promote` and `/speckit.dm-guard.status` consume the files in `gates/`. The existence of a `.md` file is not sufficient evidence of an approved gate.

## Commands Included

| Command | Alias (skills mode) |
|---------|---------------------|
| `/speckit.dm-guard.verify` | `speckit-verify` |
| `/speckit.dm-guard.security` | `speckit-security` |
| `/speckit.dm-guard.review` | `speckit-review` |
| `/speckit.dm-guard.docs` | `speckit-docs` |
| `/speckit.dm-guard.promote` | `speckit-promote` |
| `/speckit.dm-guard.status` | `speckit-status` |
| `/speckit.dm-guard.retro` | `speckit-retro` |

The base Spec Kit commands (`/speckit.specify`, `/speckit.plan`, `/speckit.implement`, etc.) are provided by the official Spec Kit and are not part of this extension.

## License

MIT — DM Tecnologia
