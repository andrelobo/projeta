# Relatório de consistência e lacunas — PRD 1.0

**Data:** 2026-11-05
**Papel:** System Architect
**Objetivo:** confrontar o PRD com a arquitetura proposta, identificar contradições, lacunas e decisões derivadas antes de qualquer implementação.

> Nada neste relatório altera o PRD. Cada item indica **onde** a questão foi tratada (spec/ADR), **como** foi resolvida ou **qual decisão** continua aberta.

---

## 1. Resumo

| Classe | Quantidade |
|:--|--:|
| Requisitos funcionais (RF) | 75 |
| Regras de negócio (RB) | 20 |
| Requisitos não funcionais (RNF) | 22 |
| Casos de uso de IA (AI) | 10 |
| Contradições/ambiguidades catalogadas | 26 |
| Lacunas de modelo/entidade | 9 |
| Lacunas de segurança/LGPD/operação | 13 |
| Assunções explícitas (ASM) | 9 |
| Decisões abertas (OPEN) | 50 |

**Cobertura:** todos os 75 RF estão mapeados para uma spec (`docs/specifications/README.md` §2). Todos os 20 RB têm local de implementação (`DOMAIN_MODEL` §9). Todos os 22 RNF têm tratamento em SPEC-000 ou na spec do módulo.

---

## 2. Contradições e ambiguidades do PRD

| ID | Onde | Questão | Resolução | Status |
|:--|:--|:--|:--|:--|
| C-01 | §10 API | rotas com `{id}` apareceram sem placeholder | normalizadas na transcrição; sem mudança semântica | resolvido |
| C-02 | RF-004 × M03 | "perfil de usuário" (conta) e "perfil profissional" colidem em `/profiles/me` | `/users/me` = conta; `/profiles/me` = perfil profissional | resolvido |
| C-03 | RF-033 × RF-038 | enum único não expressa "aberto", "encerrado" e "verificação pendente" | `status` + `verificationStatus` + `effectiveStatus` (ADR-016) | resolvido |
| C-04 | §5 × §17 | M08 (orçamento) e M09 (cronograma) constam como "MVP", mas o roadmap os coloca na Fase 2 | specs marcadas Fase 2; feature gate por plano | **OPEN-049** |
| C-05 | RF-071/RF-081 | "etapa vinculada" e "organizar por etapas" sem entidade `stage` | `stage` como string versionada no projeto; entidade dedicada só se necessário | **OPEN-004** |
| C-06 | §10 Budgets × ADR-017 | orçamento é criado em `/projects/{id}/budgets`, mas pertence à proposta | `proposalId` opcional no orçamento; quando existe, escopo é a proposta | resolvido |
| C-07 | RF-068 | `project_opportunities` não tem semântica definida | interpretada como **Proposta** (ADR-017) | resolvido |
| C-08 | RF-066 × RF-068 | não está claro se a versão é do projeto ou da proposta | versão com `projectId` + `proposalId?` (ADR-017) | resolvido |
| C-09 | RF-036 | favoritos sem coleção no §8.1 | `opportunity_favorites` (SPEC-004) | resolvido |
| C-10 | RF-037 | alertas sem entidade | `saved_searches` + job (SPEC-004/011) | resolvido |
| C-11 | RF-094 | "modelos versionados" sem entidade | `document_templates` (SPEC-010) | resolvido |
| C-12 | RF-113 | webhooks sem armazenamento de evento | `billing_events` (SPEC-012) | resolvido |
| C-13 | RB-010 | "referência às entradas" sem campo | `provenance` em versões + `ai_interactions` (SPEC-006/008) | resolvido |
| C-14 | RF-030 | `geographicScope` texto livre × filtros por UF/município | `geo { scope, country, state, city }` estruturado | resolvido |
| C-15 | RF-030 | `eligibleApplicantTypes` sem enum | enum `INDIVIDUAL/MEI/COMPANY/NONPROFIT/COLLECTIVE/PUBLIC` (SPEC-003) | resolvido |
| C-16 | RF-032 | filtro "modalidade" sem vocabulário | campo `modality` com vocabulário a definir | **OPEN-016** |
| C-17 | RF-005 | exclusão de conta × retenção legal indefinida | soft delete + anonimização após retenção | **OPEN-023** |
| C-18 | RF-012 | papéis sem matriz de capacidades | matriz em SPEC-002 §4 | resolvido |
| C-19 | RF-069 × RF-065 | "pode editar tudo" × "submetido imutável" | rascunho editável (`draft`) separado da versão congelada | resolvido |
| C-20 | §2.2 × J05 | "sem submissão automática" × "registrar submissão" | API apenas **registra** o ato do usuário; não envia a portal | resolvido |
| C-21 | RF-102 × Fase 2 | tarefas no MVP × cronograma na Fase 2 | tarefa simplificada no MVP; dependências/etapas na Fase 2 | resolvido |
| C-22 | RF-090/RF-040 | storage sem provedor nem limite | provedor em aberto; limite padrão 20 MB | **OPEN-003/020** |
| C-23 | RF-033 | `CLOSED` persistido × fechamento automático | job marca `CLOSED` com motivo; leitura deriva de qualquer forma | resolvido |
| C-24 | RF-110 × §1 | "Freemium" sem definição de trial | `TRIAL` existe no enum; política comercial em aberto | **OPEN-042** |
| C-25 | RF-101 × RF-068 | status de projeto e de proposta se sobrepõem quando há várias propostas | status da proposta é a fonte de verdade; status do projeto agrega pela mais avançada | resolvido |
| C-26 | PRD §9.1 × decisão do owner | PRD define monorepo; owner decidiu repositórios separados no GitHub | Emenda 01 (`PRD_ERRATA.md`) + ADR-021; SPEC-000 ajustada | resolvido |

---

## 3. Lacunas de modelo de dados (coleções ausentes no §8.1)

| # | Coleção | Necessária para | Onde foi definida |
|:--|:--|:--|:--|
| L-01 | `opportunity_favorites` | RF-036 | SPEC-004 |
| L-02 | `saved_searches` | RF-037 | SPEC-004/011 |
| L-03 | `document_templates` | RF-094 | SPEC-010 |
| L-04 | `billing_events` | RF-113 | SPEC-012 |
| L-05 | `prompt_templates` | RB-010, §7 | SPEC-006 |
| L-06 | `notification_preferences` / `notification_deliveries` | RF-037/RF-103 | SPEC-011 |
| L-07 | `membership_invites` | RF-011 | SPEC-002 |
| L-08 | `user_sessions` / `email_tokens` | RF-001..003 | SPEC-001 |
| L-09 | `outbox` / `idempotency_keys` / `feature_flags` / `schema_migrations` | RNF-019, infra | SPEC-000 |

> São extensões **derivadas** dos requisitos, não requisitos novos. Todas registradas nas specs correspondentes.

---

## 4. Lacunas de segurança, LGPD e operação

| # | Lacuna no PRD | Tratamento |
|:--|:--|:--|
| S-01 | política de senha não definida | mínimo 10 + blocklist (SPEC-001, ASM-002) |
| S-02 | CSRF não citado | ADR-014 + SPEC-001 |
| S-03 | revogação de sessão não detalhada | famílias de refresh + revoke-all (SPEC-001) |
| S-04 | verificação de e-mail não obrigatória | exigida para ações pagas (SPEC-001) |
| S-05 | portabilidade LGPD não citada | inclusão no roadmap de privacidade (`OPEN-023`) |
| S-06 | retenção por categoria indefinida | `OPEN-023`, `OPEN-045` |
| S-07 | resposta a incidente não citada | runbook na Fase 0 (SPEC-000) |
| S-08 | gestão de segredos não citada | cofre + proibição de log (SPEC-000) |
| S-09 | SSRF em conectores não citado | allowlist de domínio, bloqueio de IP privado (SYSTEM_DESIGN §12) |
| S-10 | RPO/RTO de backup não definidos | **OPEN-050** |
| S-11 | acessibilidade sem nível | WCAG 2.1 AA (ASM-001) |
| S-12 | DPIA para HEALTH | exigida antes da Fase 3 (ADR-007) |
| S-13 | repositórios públicos expõem código e roadmap | licença (`OPEN-047`), secret scanning/push protection, revisão do que entra em `docs/` (ADR-021) |

---

## 5. Assunções explícitas (ASM)

| ID | Assunção | Onde |
|:--|:--|:--|
| ASM-001 | acessibilidade alvo = WCAG 2.1 AA | SPEC-000 |
| ASM-002 | senha mínima de 10 caracteres com blocklist | SPEC-001 |
| ASM-003 | fuso padrão de exibição = America/Manaus | AGENTS.md, SPEC-001 |
| ASM-004 | upload padrão de 20 MB (configurável) | SPEC-010 |
| ASM-005 | limite padrão de 300 páginas por PDF | SPEC-005 |
| ASM-006 | access 15 min / refresh 30 dias / token de e-mail 30 min | SPEC-001 |
| ASM-007 | janela de verificação de oportunidade = 30 dias (proposta) | SPEC-004 / OPEN-018 |
| ASM-008 | limites numéricos dos planos não definidos nesta fase | SPEC-012 |
| ASM-009 | proposta = project × opportunity; orçamento segue a proposta | ADR-017 |

Assunções **não** são requisitos: qualquer uma pode ser alterada por decisão registrada.

---

## 6. Cobertura de requisitos

```
M01 RF-001..005  → SPEC-001
M02 RF-010..013  → SPEC-002
M03 RF-020..023  → SPEC-003
M04 RF-030..039  → SPEC-004
M05 RF-040..046  → SPEC-005
M06 RF-050..053  → SPEC-007
M07 RF-060..069  → SPEC-008
M08 RF-070..075  → SPEC-009 (Fase 2)
M09 RF-080..083  → SPEC-009 (Fase 2)
M10 RF-090..094  → SPEC-010
M11 RF-100..103  → SPEC-011
M12 RF-110..115  → SPEC-012
M13 RF-120..125  → SPEC-013
AI-001..010      → SPEC-006 + specs de uso
RNF-001..022     → SPEC-000 + specs de módulo
RB-001..020      → DOMAIN_MODEL §9
```

Nenhum RF ficou órfão. Nenhuma spec cobre requisito inexistente.

---

## 7. O que foi decidido sem base explícita no PRD

Itens marcados como **proposta** e que exigem aprovação:

1. OwnerRef/tenantKey (ADR-015).
2. Status efetivo + verificação (ADR-016).
3. Proposta × versão (ADR-017).
4. Contratos Zod/OpenAPI (ADR-012).
5. Cookie + CSRF (ADR-014).
6. Money/Decimal128 (ADR-013).
7. Storage S3 assinado (ADR-018).
8. Outbox/DLQ (ADR-019).
9. Prompt registry/custo (ADR-020).
10. Polyrepo no GitHub (ADR-021) e deploy Vercel + VPS (ADR-022).
11. Coleções derivadas (L-01..L-09).
12. Tarefa simplificada no MVP (C-21).
13. Orçamento na proposta quando vinculado (C-06).

Nenhum desses itens altera requisito de negócio; todos são meios de implementação ou detalhamento necessário.

---

## 8. Riscos de interpretação

| Risco | Consequência | Mitigação |
|:--|:--|:--|
| Tratar "MVP" do §5 como tudo obrigatório | escopo explode | §17 prevalece; Fase 2 explícita |
| Confundir "verificado" com "aberto" | informação incorreta ao usuário | ADR-016 + testes |
| Deixar IA decidir elegibilidade | viola RF-053 | engine determinística + regra TEXT_MANUAL |
| Versionar projeto em vez de proposta | histórico errado por edital | ADR-017 |
| Publicar sem fonte | viola RB-001 | invariante no domínio + teste |
| Assumir que todo portal tem API | coleta ilegal/frágil | ADR-008 + OPEN-019 |

---

## 9. Próximos passos de validação

1. Ratificar a Emenda 01 (`PRD_ERRATA.md`) e as decisões propostas (ADRs 012–023).
2. Resolver as decisões abertas bloqueantes por milestone (`OPEN_DECISIONS.md`).
3. Aprovar as specs em blocos, começando por SPEC-000 a SPEC-004.
4. Só então iniciar M0.
