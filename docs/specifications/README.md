# Especificações — Índice e rastreabilidade

Especificações implementáveis derivadas do PRD 1.0. Cada spec é a unidade de aprovação e de implementação. **Nenhuma linha de código deve existir sem spec aprovada.**

- Status atual: **SPEC-000 a SPEC-004 aprovadas** (2026-11-05); **SPEC-005 a SPEC-013 em Rascunho**.
- Template: [`_TEMPLATE.md`](_TEMPLATE.md)
- Modelo de domínio: [`../architecture/DOMAIN_MODEL.md`](../architecture/DOMAIN_MODEL.md)
- Arquitetura: [`../architecture/SYSTEM_DESIGN.md`](../architecture/SYSTEM_DESIGN.md)

## 1. Lista de specs

| ID | Título | Fase | Depende de | Requisitos |
|:--|:--|:--|:--|:--|
| [SPEC-000](SPEC-000-platform-foundation.md) | Fundação da plataforma | 0 | — | RNF-001..022 |
| [SPEC-001](SPEC-001-identity-auth.md) | Identidade e autenticação | 1 | SPEC-000 | RF-001..005 |
| [SPEC-002](SPEC-002-organizations-tenancy.md) | Organizações, membros e tenancy | 1 | SPEC-001 | RF-010..013, RB-005, RB-006, RB-016 |
| [SPEC-003](SPEC-003-professional-profile.md) | Perfil profissional | 1 | SPEC-001, SPEC-010 | RF-020..023 |
| [SPEC-004](SPEC-004-opportunities-radar.md) | Oportunidades e Radar | 1 | SPEC-001, SPEC-002 | RF-030..039, RB-001, RB-002, RB-019 |
| [SPEC-005](SPEC-005-edital-analyzer.md) | Analisador de editais | 1 | SPEC-003, SPEC-006, SPEC-010 | RF-040..046 |
| [SPEC-006](SPEC-006-ai-platform.md) | Plataforma de IA e guardrails | 1 | SPEC-000 | AI-001..010, §7, RB-009, RB-010, RB-018 |
| [SPEC-007](SPEC-007-eligibility.md) | Motor de elegibilidade | 1 | SPEC-003, SPEC-004, SPEC-005 | RF-050..053 |
| [SPEC-008](SPEC-008-projects-versioning.md) | Projetos, propostas e versionamento | 1 | SPEC-003, SPEC-004, SPEC-006 | RF-060..069, RB-011 |
| [SPEC-009](SPEC-009-budgets-schedule.md) | Orçamento e cronograma | 2 | SPEC-008 | RF-070..083 |
| [SPEC-010](SPEC-010-documents-export.md) | Documentos e exportação | 1 | SPEC-000, SPEC-002 | RF-090..094, RB-007, RNF-020 |
| [SPEC-011](SPEC-011-dashboard-alerts.md) | Dashboard, alertas e notificações | 1 | SPEC-004, SPEC-008 | RF-037, RF-100..103 |
| [SPEC-012](SPEC-012-billing-entitlements.md) | Billing, planos e entitlements | 1 | SPEC-001, SPEC-002 | RF-110..115, RB-012, RB-013, RB-017 |
| [SPEC-013](SPEC-013-admin-audit.md) | Administração, fontes e auditoria | 1 | SPEC-001, SPEC-004, SPEC-012 | RF-120..125, RB-016 |

## 2. Matriz requisito → spec

| Requisitos | Spec |
|:--|:--|
| RF-001 a RF-005 | SPEC-001 |
| RF-010 a RF-013 | SPEC-002 |
| RF-020 a RF-023 | SPEC-003 |
| RF-030 a RF-039 | SPEC-004 |
| RF-040 a RF-046 | SPEC-005 |
| RF-050 a RF-053 | SPEC-007 |
| RF-060 a RF-069 | SPEC-008 |
| RF-070 a RF-075 | SPEC-009 |
| RF-080 a RF-083 | SPEC-009 |
| RF-090 a RF-094 | SPEC-010 |
| RF-100 a RF-103 | SPEC-011 |
| RF-110 a RF-115 | SPEC-012 |
| RF-120 a RF-125 | SPEC-013 |
| AI-001 a AI-010 | SPEC-006 (execução) + spec do caso de uso |
| RNF-001 a RNF-022 | SPEC-000 (transversal) |
| RB-001 a RB-020 | distribuídas; ver DOMAIN_MODEL §9 |

## 3. Ordem de implementação (resumo)

```
SPEC-000 ──► SPEC-001 ──► SPEC-002 ──► SPEC-003 ──► SPEC-010
                                               │
                                               ├──► SPEC-004 ──► SPEC-006 ──► SPEC-005 ──► SPEC-007
                                               │
                                               └──► SPEC-008 ──► SPEC-011
                                                        │
                                                        └──► SPEC-009 (Fase 2)
                                               SPEC-012 (transversal ao MVP)
                                               SPEC-013 (admin mínimo no MVP)
```

Dependências detalhadas e marcos: [`../implementation/IMPLEMENTATION_PLAN.md`](../implementation/IMPLEMENTATION_PLAN.md).

## 4. Convenções das specs

- **RF cobertos** sempre citados; requisito sem spec é dívida de planejamento.
- **Contratos** em Zod (fonte de verdade) e exemplos JSON.
- **Critérios de aceite** em Gherkin, testáveis.
- **Segurança** explícita: capacidade, tenant, dados sensíveis.
- **Erros** seguem Problem Details (ADR-012).
- **DoD** referencia o checklist global e acrescenta itens específicos.
- Toda mudança de spec gera entrada no changelog do arquivo.
