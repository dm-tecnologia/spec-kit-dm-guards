# Spec Kit - DM Guards

Extensão autocontida para o fluxo de desenvolvimento Spec Kit (Specification-Driven Development). Complementa o Spec Kit oficial do GitHub com gates de verificação, revisão independente, segurança, documentação, promoção, status e retrospectiva.

**Versão**: 1.0.0

**Autor**: DM Tecnologia

**Compatibilidade**: Requer projeto inicializado com Spec Kit (`.specify/`) e um agente suportado por `specify integration`

> **English**: See [README.md](README.md)

---

## Estrutura

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

> O diretório `skills/` contém os mesmos comandos no formato skills-mode para integrações que usam skills em vez de slash commands. Ambos os formatos são mantidos em sincronia.

## Fluxo Completo

```text
FOUNDATION
  /speckit.constitution
      ↓
SPECIFICATION
  /speckit.specify → /speckit.clarify → /speckit.checklist
      ↓
DESIGN
  /speckit.plan → /speckit.tasks → /speckit.taskstoissues (opcional)
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

## Comandos

### Críticos

| Comando | O que faz | Por que é necessário |
|---------|-----------|---------------------|
| `/speckit.dm-guard.verify` | Cross-reference spec/tasks × código real. Detecta tasks marcadas como feitas sem evidência, drift arquitetural e gaps por user story. | `[X]` em `tasks.md` não prova que a implementação existe. |
| `/speckit.dm-guard.review` | Gate de code review com security scan, baseline-aware checks e revisão independente local. | Nenhum agente deve revisar o próprio código sem fresh context. |
| `/speckit.dm-guard.promote` | Gate final: valida gates anteriores, gera commit convencional, pede permissão e abre PR com artefatos linkados. | Fecha rastreabilidade entre spec, código, gates e PR. |

### Altos

| Comando | O que faz |
|---------|-----------|
| `/speckit.dm-guard.security` | Gera checklist OWASP, valida auth/authz, dados sensíveis e secrets. |
| `/speckit.dm-guard.status` | Dashboard de lifecycle por feature, fase, gate e blocker. |
| `/speckit.dm-guard.retro` | Captura lições aprendidas, decisões arquiteturais e propostas de constitution. |

### Qualidade

| Comando | O que faz |
|---------|-----------|
| `/speckit.dm-guard.docs` | Verifica sincronização de docs, contratos, context files, env examples e i18n. |

### Interno

`speckit-independent-review` é uma skill interna invocada pelo `/speckit.dm-guard.review`. Recebe diff, contexto mínimo e resultados de scans; retorna veredito estruturado sem depender de skills externas. Não possui comando standalone.

## Uso

### Instalar Como Extensão Spec Kit

```bash
# A partir do clone local
specify extension add --dev /path/to/spec-kit-dm-guards

# A partir do GitHub
specify extension add spec-kit-dm-guards --from https://github.com/dm-tecnologia/spec-kit-dm-guards
```

Os comandos são registrados automaticamente no agente configurado.

### Fluxo Típico De Uma Feature

```bash
# 1. Especificação
/speckit.specify "Adicionar autenticação via WhatsApp"
/speckit.clarify
/speckit.checklist

# 2. Design
/speckit.plan
/speckit.tasks

# 3. Gate de análise
/speckit.analyze

# 4. Implementação
/speckit.implement

# 5. Gates DM Guard
/speckit.dm-guard.verify
/speckit.dm-guard.security
/speckit.dm-guard.review
/speckit.dm-guard.docs

# 6. Entrega
/speckit.dm-guard.promote

# 7. Governança
/speckit.dm-guard.retro
/speckit.dm-guard.status
```

## Artefatos De Gate

Cada gate gera um relatório Markdown e um status JSON estruturado conforme `gate-status-schema.md`.

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

`/speckit.dm-guard.promote` e `/speckit.dm-guard.status` consomem os arquivos em `gates/`. A existência de um `.md` não é evidência suficiente de gate aprovado.

## Comandos Incluídos

| Comando | Alias (skills mode) |
|---------|---------------------|
| `/speckit.dm-guard.verify` | `speckit-verify` |
| `/speckit.dm-guard.security` | `speckit-security` |
| `/speckit.dm-guard.review` | `speckit-review` |
| `/speckit.dm-guard.docs` | `speckit-docs` |
| `/speckit.dm-guard.promote` | `speckit-promote` |
| `/speckit.dm-guard.status` | `speckit-status` |
| `/speckit.dm-guard.retro` | `speckit-retro` |

Os comandos base do Spec Kit (`/speckit.specify`, `/speckit.plan`, `/speckit.implement`, etc.) são fornecidos pelo Spec Kit oficial e não fazem parte desta extensão.

## Licença

MIT — DM Tecnologia
