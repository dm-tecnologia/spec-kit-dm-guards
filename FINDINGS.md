# Findings — Spec Kit Agent Skills

## Context

This document consolidates the findings from the initial evaluation of the local skills and documents against the official Spec Kit (`github/spec-kit`).

## Resolved

- ✅ Hermes references removed; package targets Spec Kit agents via skills mode and extension.
- ✅ `speckit-independent-review` created as self-contained review skill.
- ✅ External review dependencies removed from `speckit-review`.
- ✅ `gate-status-schema.md` created as common contract.
- ✅ `speckit-verify` made read-only by default.
- ✅ `speckit-promote` validates gates via structured JSON, not Markdown existence.
- ✅ `specify extension add --dev` packaging with `extension.yml`, `commands/`, hooks.
- ✅ `grep -P` patterns replaced with portable commands in `speckit-docs`.
- ✅ Security gate supports pre-plan and post-implementation modes.
- ✅ Extension ID `dm-guard`; commands follow `speckit.dm-guard.*` pattern.
- ✅ Bilingual docs: EN (README.md, speckit-flow.md) + PT (README.pt.md, speckit-flow.pt.md).
- ✅ Script paths corrected in all 4 command files.
- ✅ Retro SKILL.md fallback scans for `gates/promote.json` (JSON), not Markdown.
- ✅ `promote.md` added to all artifact tree diagrams.
- ✅ `speckit-agent-context-update` removed from docs SKILL.md related_skills.
- ✅ `speckit-flow.pt.md` back-link to English version added.
- ✅ `README.pt.md` structure tree includes `README.pt.md` and `speckit-flow.pt.md`.
- ✅ Docs gate validates prior gate statuses (review.json) before proceeding.
- ✅ Review and promote commands reference `gate-status-schema.md`.
- ✅ Security exception example includes `approved_by` and `approved_at` fields.

## Flow Validation — Passed

- Gate dependencies correct: verify → security → review → docs → promote.
- `promote` correctly validates gates via `gates/*.json`.
- `status` and `retro` consume structured gate JSON.
- `speckit-independent-review` is internal (used by review, no standalone command).
- Hooks consistent with flow.
- Gate status schema matches what each skill writes.
- `check-prerequisites.sh` paths correct across all files.
