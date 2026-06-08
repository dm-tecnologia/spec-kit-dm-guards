# v1.0.0 — Initial Release

First stable release of **Spec Kit DM Guards**, a post-implementation gate extension for the [Spec Kit](https://github.com/github/spec-kit) SDD flow.

## What it does

Adds 7 structured gates after `/speckit.implement` to prevent incomplete, insecure, undocumented, or unreviewed code from reaching production.

```
/speckit.implement
     ↓
/speckit.dm-guard.verify    ← spec/tasks × actual code
     ↓
/speckit.dm-guard.security  ← OWASP + secrets + auth/authz
     ↓
/speckit.dm-guard.review    ← static scan + independent review
     ↓
/speckit.dm-guard.docs      ← contracts, i18n, context files sync
     ↓
/speckit.dm-guard.promote   ← validate gates → commit → push → PR
     ↓
/speckit.dm-guard.retro     ← lessons, ADRs, constitution proposals
/speckit.dm-guard.status    ← feature dashboard (MD + HTML)
```

## Included components

| Component | Description |
|---|---|
| **7 gate commands** | verify, security, review, docs, promote, status, retro |
| **Independent review skill** | `speckit-independent-review` — self-contained reviewer, no external dependencies |
| **Gate status schema** | `gate-status-schema.md` — structured JSON contract (`gates/*.json`) |
| **Extension packaging** | `extension.yml` + `commands/` — installable via `specify extension add` |
| **Configuration** | `config-template.yml` — per-project gate policy, docs targets, thresholds |
| **Status dashboard** | `skills/speckit-status/status-template.html` — self-contained HTML + Markdown output |
| **Bilingual docs** | README + speckit-flow in English and Portuguese |

## Key design decisions

- **Fail-closed**: gates block by default; exceptions require explicit approval
- **Structured evidence**: every gate writes `gates/<gate>.json`; promotion reads JSON, not Markdown existence
- **Verify is read-only**: never mutates `tasks.md` without user approval
- **No worktree mutation**: review gate never uses `git stash`
- **Extension ID**: `dm-guard` → commands follow `speckit.dm-guard.*` pattern
- **Agent portability**: no dependency on specific agents or unbundled skills

## Installation

```bash
specify extension add spec-kit-dm-guards --from https://github.com/dm-tecnologia/spec-kit-dm-guards
```

Requires: Spec Kit ≥ 0.8.0
