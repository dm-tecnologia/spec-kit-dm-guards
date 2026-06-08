# Spec Kit Development Flow — DM Gates

> **Versão**: 1.0.0 | **Última atualização**: 2026-06-08  
> Fluxo Spec Kit com gates complementares para verificação, segurança, revisão independente, documentação, promoção e governança.
>
> **English**: See [speckit-flow.md](speckit-flow.md)

---

## Fluxo Completo

```text
FOUNDATION
  /speckit.constitution
      ↓
SPECIFICATION
  /speckit.specify
      ↓
  /speckit.clarify       (até 5 perguntas estruturadas quando necessário)
      ↓
  /speckit.checklist     (qualidade e completude de requisitos)
      ↓
DESIGN
  /speckit.plan
      ↓
  /speckit.tasks
      ↓
  /speckit.taskstoissues (opcional)
      ↓
ANALYSIS GATE
  /speckit.analyze       (spec.md × plan.md × tasks.md)
      ↓
IMPLEMENTATION
  /speckit.implement
      ↓
VERIFICATION GATES
  /speckit.dm-guard.verify         (spec/tasks × código real)
      ↓
  /speckit.dm-guard.security       (security requirements + code scan)
      ↓
  /speckit.dm-guard.review         (review gate + speckit-independent-review)
      ↓
  /speckit.dm-guard.docs           (docs, contratos e context files sincronizados)
      ↓
DELIVERY
  /speckit.dm-guard.promote        (valida gates, commit aprovado, push/PR aprovados)
      ↓
GOVERNANCE
  /speckit.dm-guard.retro          (lições, ADRs, propostas de constitution)
  /speckit.dm-guard.status         (dashboard de features e gates)
```

## Tabela De Gates E Decisões

| Fase | Skill/Comando | Gate | Se PASS | Se FAIL |
|------|---------------|------|---------|---------|
| Foundation | `/speckit.constitution` | Princípios do projeto definidos | Seguir para spec | Corrigir constitution antes de iniciar feature |
| Specification | `/speckit.specify` | `spec.md` criado sem implementação prematura | Rodar clarify/checklist | Refinar spec |
| Specification | `/speckit.clarify` | Ambiguidades críticas resolvidas ou explicitamente deferidas | Rodar checklist | Responder perguntas pendentes |
| Specification | `/speckit.checklist` | Requisitos completos, claros e testáveis | Seguir para plan | Corrigir spec |
| Design | `/speckit.plan` | Constitution check + design artifacts | Seguir para tasks | Corrigir plano |
| Design | `/speckit.tasks` | `tasks.md` executável, ordenado por user story | Rodar analyze | Regenerar tasks |
| Analysis | `/speckit.analyze` | Consistência spec × plan × tasks | Seguir para implement | Corrigir inconsistências |
| Implementation | `/speckit.implement` | Tasks executadas e marcadas `[X]` | Rodar verify | Corrigir implementação |
| Verify | `/speckit.dm-guard.verify` | Sem P1 gaps e cobertura aceitável por user story | Rodar security | Corrigir gaps ou aprovar exceção |
| Security | `/speckit.dm-guard.security` | Sem secrets, auth/authz adequado e riscos bloqueantes tratados | Rodar review | Corrigir gaps de segurança |
| Review | `/speckit.dm-guard.review` | 0 findings críticos/altos bloqueantes | Rodar docs | Corrigir findings bloqueantes |
| Docs | `/speckit.dm-guard.docs` | Docs/contratos/context files sincronizados ou não aplicáveis | Rodar promote | Corrigir gaps críticos de docs |
| Promote | `/speckit.dm-guard.promote` | Gates obrigatórios aprovados e usuário aprovou ações Git | Commit/push/PR | Corrigir gate pendente |

## Política De Gates

- Gates obrigatórios padrão: `verify`, `security`, `review`, `docs` e `promote`.
- Um gate pode retornar `not-applicable` quando a skill justificar por evidência que o gate não se aplica à feature.
- `promote` não deve aceitar apenas a existência de arquivos Markdown; deve ler os JSONs em `FEATURE_DIR/gates/`.
- Warnings só permitem promoção quando não há findings bloqueantes e a justificativa está registrada no status do gate.
- A política pode ser configurada por projeto no futuro, mas o default é fail-closed.

## Artefatos Por Feature

```text
specs/<feature>/
├── spec.md                    ← Especificação (/speckit.specify)
├── plan.md                    ← Plano técnico (/speckit.plan)
├── research.md                ← Pesquisa técnica (plan phase 0)
├── data-model.md              ← Modelo de dados (plan phase 1)
├── quickstart.md              ← Guia de validação (plan phase 1)
├── tasks.md                   ← Lista de tasks (/speckit.tasks)
├── contracts/                 ← Contratos de interface
├── checklists/
│   ├── requirements.md        ← Checklist de requisitos (/speckit.checklist)
│   └── security.md            ← Checklist OWASP (/speckit.dm-guard.security)
├── verify.md                  ← Relatório spec/tasks × código
├── review.md                  ← Relatório de code review
├── docs-check.md              ← Relatório de documentação
├── promote.md                 ← Relatório de promoção
├── retro.md                   ← Retrospectiva
└── gates/
    ├── verify.json            ← Status estruturado do gate verify
    ├── security.json          ← Status estruturado do gate security
    ├── review.json            ← Status estruturado do gate review
    ├── docs.json              ← Status estruturado do gate docs
    └── promote.json           ← Status estruturado do gate promote
```

O schema dos arquivos em `gates/` está documentado em `gate-status-schema.md`.

## Propriedades Do Fluxo

### Invariantes

1. **Specs são a fonte primária de intenção**: requisitos e decisões devem evoluir nos artefatos Spec Kit.
2. **Tasks são plano executável, não prova de entrega**: `[X]` em `tasks.md` precisa de evidência no código.
3. **Verify é read-only por padrão**: reconciliações em `tasks.md` exigem aprovação explícita.
4. **Gate Markdown não é suficiente**: decisões de promoção usam status estruturado.
5. **Nunca commitar, pushar ou abrir PR sem permissão explícita**.
6. **Nenhum agente revisa o próprio código sem fresh context**: `/speckit.dm-guard.review` usa `speckit-independent-review` como revisão independente local.

### Paralelismo

- Dentro de uma feature, tasks marcadas `[P]` podem rodar em paralelo quando não tocam os mesmos arquivos.
- Gates pós-implementação rodam sequencialmente por padrão porque cada gate consome evidência do anterior.
- A revisão independente pode usar subagent/fresh context quando o agente suportar; caso contrário, deve operar em modo fallback seguindo `speckit-independent-review`.

### Loops De Feedback

- `analyze → plan/specify`: inconsistências entre spec, plan e tasks voltam para os artefatos corretos.
- `verify → implement`: gaps de implementação voltam para código ou para uma decisão explícita de escopo.
- `security → specify/plan/implement`: requisitos de segurança ausentes voltam para spec/plan; vulnerabilidades voltam para código.
- `review → implement`: findings bloqueantes voltam para correção mínima e re-review.
- `docs → plan/docs`: contratos e docs divergentes voltam para atualização documental ou correção de implementação.
- `retro → constitution`: aprendizados viram propostas aprováveis de constitution/ADR.

## Comparação: Spec Kit Original Vs DM Gates

| Aspecto | Spec Kit Original | DM Gates |
|---------|-------------------|----------|
| Especificação | Sim | Usa fluxo oficial |
| Clarificação | Sim | Usa fluxo oficial |
| Checklist de requisitos | Sim | Usa fluxo oficial |
| Design | Sim | Usa fluxo oficial |
| Tasks | Sim | Usa fluxo oficial |
| Análise spec × plan × tasks | Sim | Usa fluxo oficial |
| Implementação | Sim | Usa fluxo oficial |
| Verificação spec/tasks × código | Não | `/speckit.dm-guard.verify` |
| Security gate | Não | `/speckit.dm-guard.security` |
| Code review independente | Manual/ad hoc | `/speckit.dm-guard.review` + `speckit-independent-review` |
| Documentação sync | Não | `/speckit.dm-guard.docs` |
| Commit/PR estruturado | Manual | `/speckit.dm-guard.promote` |
| Dashboard de gates | Não | `/speckit.dm-guard.status` |
| Retrospectiva/feedback | Não | `/speckit.dm-guard.retro` |

## Princípios De Design

1. **Fail-closed**: na dúvida, bloqueie ou exija aprovação explícita.
2. **Evidence over claims**: `[X]` em `tasks.md` não é evidência; código, testes, contratos e diffs são.
3. **Structured gates**: todo gate deve produzir status JSON consumível por outras skills.
4. **Fresh eyes**: revisão independente deve acontecer com contexto separado quando possível.
5. **Permission before action**: commit, push, PR e alteração de tasks exigem permissão.
6. **Docs with code**: docs e contratos fazem parte da definição de pronto.
7. **Agent portability**: skills não devem depender de um agente específico ou de skills externas não empacotadas neste repositório.

## Instalação

Este pacote funciona como extensão Spec Kit e como skills independentes.

### Extensão Spec Kit

```bash
specify extension add spec-kit-dm-guards --from https://github.com/dm-tecnologia/spec-kit-dm-guards
# ou a partir do clone local
specify extension add --dev /path/to/spec-kit-dm-guards
```

Comandos registrados: `/speckit.dm-guard.verify`, `/speckit.dm-guard.security`, `/speckit.dm-guard.review`, `/speckit.dm-guard.docs`, `/speckit.dm-guard.promote`, `/speckit.dm-guard.status`, `/speckit.dm-guard.retro`.

### Skills Mode

Copie ou faça symlink da pasta `skills/` para o diretório de skills do agente configurado.

### Hooks

A extensão registra hooks opcionais após fases do Spec Kit (configure via `config.yml`):
- `after_implement` → `speckit.dm-guard.verify`
- `after_verify` → `speckit.dm-guard.security`
- `after_security` → `speckit.dm-guard.review`
- `after_review` → `speckit.dm-guard.docs`
- `after_docs` → `speckit.dm-guard.promote`

### Aliases (Skills Mode)

| Comando | Alias skills-mode |
|---------|-------------------|
| `/speckit.dm-guard.verify` | `speckit-verify` |
| `/speckit.dm-guard.security` | `speckit-security` |
| `/speckit.dm-guard.review` | `speckit-review` |
| `/speckit.dm-guard.docs` | `speckit-docs` |
| `/speckit.dm-guard.promote` | `speckit-promote` |
| `/speckit.dm-guard.status` | `speckit-status` |
| `/speckit.dm-guard.retro` | `speckit-retro` |
