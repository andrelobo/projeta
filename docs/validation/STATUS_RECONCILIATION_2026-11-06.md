# Reconciliação de status — 2026-11-06

**Papel:** System Architect
**Objetivo:** eliminar a divergência entre documentos que declaravam o plano/specs como não aprovados e os registros formais de aprovação.
**Escopo:** documentação e licença. Nenhum código, dependência, commit ou deploy.

> **Atualização posterior (2026-11-06):** após as decisões registradas, o M0.1 e o M0.2 foram executados nos repositórios de aplicação. Evidências em `M0_PROGRESS.md`. As seções abaixo preservam o estado da reconciliação documental.

---

## 1. Fonte de verdade adotada

| Item | Estado vigente | Evidência |
|:--|:--|:--|
| Plano de implementação | Aprovado em 2026-11-05 | `IMPLEMENTATION_PLAN.md`, `APPROVALS.md` |
| ADR-001 a ADR-010 | Aceitos (vieram do PRD) | `docs/architecture/ADR/` |
| ADR-011 | Rejeitado | `ADR-011-monorepo-pnpm.md` |
| ADR-012 a ADR-025 | Aceitos | `docs/architecture/ADR/`, `APPROVALS.md` |
| SPEC-000 a SPEC-004 | Aprovadas em 2026-11-05 | cabeçalho das specs, `APPROVALS.md` |
| SPEC-005 a SPEC-013 | Rascunho | cabeçalho das specs |
| Código de aplicação | Fundação M0.1/M0.2 (sem funcionalidade de produto) | `projeta-api/`, `projeta-web/`, `M0_PROGRESS.md` |
| Licença | Apache-2.0 nos três repositórios | `LICENSE` em `projeta/`, `projeta-api/`, `projeta-web/` |

**Gate do M0:** plano e primeiro bloco de specs aprovados; ainda pendentes **OPEN-002** (runner de migração), **OPEN-008** (observabilidade) e a confirmação de que não há dado real de cliente no ambiente de desenvolvimento.

---

## 2. O que foi corrigido nesta data

| Arquivo | Correção |
|:--|:--|
| `CONTEXT.md` | estado, contagem de ADRs, status das specs e próximo passo |
| `AGENTS.md` | estado da fase e status das specs (sem alterar regras) |
| `README.md` | removida a afirmação de que a implementação depende de aprovação do plano |
| `docs/specifications/README.md` | status: 5 aprovadas, 9 em rascunho |
| `docs/implementation/IMPLEMENTATION_PLAN.md` | checkboxes de aprovação e licença; nota de status |
| `docs/implementation/APPROVALS.md` | pendências de OPEN-053, OPEN-054 e confirmação de dados reais |
| `docs/implementation/REPOSITORY_INSPECTION.md` | marcado como relatório histórico; contagem de ADRs |
| `docs/validation/OPEN_DECISIONS.md` | OPEN-053 e nota sobre a lacuna OPEN-033 |
| `projeta-api/LICENSE`, `projeta-web/LICENSE` | Apache-2.0 aplicada |
| `projeta-api/README.md`, `projeta-web/README.md` | licença corrigida (OPEN-047 encerrada) |

---

## 3. Pendências que exigem decisão humana

### 3.1 Emenda 02 × ADR-023 — OPEN-053

- `ADR-023` está **Aceito** e o `APPROVALS.md` registra ADR-012 a ADR-025 como aprovados.
- `PRD_ERRATA.md` mantém a **Emenda 02 como proposta, pendente de aprovação**, com autoridade “System Architect”.

Não é possível resolver isso por edição unilateral: uma emenda ao PRD exige aprovação do Product Owner. As opções são:

1. **Ratificar a Emenda 02** pelo Product Owner e atualizar `PRD_ERRATA.md`; ou
2. **Suspender o ADR-023** até que a emenda seja aprovada.

Recomendação: ratificar, pois a arquitetura de autenticação cross-domain já está referenciada pelo `SYSTEM_DESIGN` e pelo `ADR-024`.

### 3.2 OPEN-002 e OPEN-008

Continuam bloqueando o M0. Recomendação já registrada em `OPEN_DECISIONS.md`:

- OPEN-002: script de migração versionado simples;
- OPEN-008: OpenTelemetry com backend gerenciado.

### 3.3 Token de leitura do GitHub Packages na Vercel

O `projeta-web` consome `@projeta/contracts` do GitHub Packages, mas o `ADR-024` só prevê `GHCR_READ_TOKEN` para a VPS. O build da Vercel também precisa de um token com `read:packages` (por exemplo, `NPM_TOKEN`/`NODE_AUTH_TOKEN`). Ação de M0, não decisão de produto.

### 3.4 Correções editoriais em ADRs aceitos

- `ADR-021` afirma “privados, salvo decisão em contrário” e, na mesma decisão, define os repositórios como públicos.
- Correção exige aprovação humana por se tratar de ADR aceito; pode ser feita como errata editorial ou ADR-026 de esclarecimento.

### 3.5 Ratificação dos documentos de arquitetura

`SYSTEM_DESIGN.md`, `DOMAIN_MODEL.md` e `TEST_STRATEGY.md` ainda estão marcados como “proposta / aguardando aprovação”. Eles são a base técnica das specs aprovadas, mas não há registro formal de aprovação no `APPROVALS.md`. A pendência está registrada como **OPEN-054**. Recomendação: ratificar os três em bloco, junto com OPEN-053.

### 3.6 Itens de higiene documental (P1)

- `SPEC-000` tinha dois itens numerados “10.” na seção de decisões; corrigido nesta data com entrada de changelog.
- `OPEN-033` não é utilizado; a lacuna está apenas documentada.
- O `CONSISTENCY_REPORT.md` ainda trata ADRs 012–023 como “propostas” em §7; foi adicionada uma nota de atualização.

---

## 4. Evidências

- Verificação de links internos: **0 quebrados**.
- Verificação de referências `OPEN-*`: nenhuma referência órfã, exceto a lacuna documentada `OPEN-033`.
- Verificação de status das specs: 5 aprovadas, 9 em rascunho.
- Repositórios de aplicação: no momento da reconciliação, apenas `README.md` e `LICENSE`; sem `package.json`, código, CI ou dependências.

---

## 5. Próximo passo

1. Product Owner decide OPEN-053 e OPEN-054.
2. Resolver OPEN-002 e OPEN-008.
3. Confirmar ausência de dados reais no ambiente de desenvolvimento.
4. Iniciar M0.1 (bootstrap dos repositórios) e M0.2 (config tipada).
