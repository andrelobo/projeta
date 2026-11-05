# DOMAIN MODEL — PROJETA

**Versão:** 1.0 (proposta)
**Status:** aguardando aprovação humana
**Base:** PRD 1.0 · ADR-009 (multi-tenancy) · ADR-015 (ownership) · ADR-016 (status efetivo) · ADR-017 (proposta × versão)
**Última atualização:** 2026-11-05

> Modelo conceitual e invariantes. Os schemas físicos (Mongoose) e os contratos de API derivam deste documento e de `@projeta/contracts` (publicado pelo `projeta-api`).

---

## 1. Contextos delimitados

| Contexto | Agregado raiz | Responsabilidade |
|:--|:--|:--|
| Identity | `User` | conta, credenciais, sessão, consentimentos |
| Organization | `Organization` | pessoa jurídica/coletivo, membros, papéis |
| Profile | `ProfessionalProfile` | dados profissionais, portfólio, documentos pessoais |
| Opportunity | `Opportunity` | chamadas, fontes, evidências, verificação, curadoria |
| Document | `Document` | arquivos privados, versões, classificação |
| Edital Analysis | `EditalAnalysis` | extração estruturada com evidências |
| Eligibility | `EligibilityAssessment` | avaliação determinística de requisitos |
| Project | `Project` | conteúdo autoral, seções, versões |
| Proposal | `ProjectOpportunity` | vínculo projeto × oportunidade, submissão |
| Budget | `Budget` | itens, totais, validação contra edital |
| Schedule | `ProjectActivity` | atividades, etapas, dependências |
| Billing | `Subscription` | plano, status, limites |
| Notification | `Notification` | avisos, alertas, central |
| Audit | `AuditLog` | trilha imutável |

`HEALTH` é um **contexto previsto** com as mesmas fronteiras, mas regras de elegibilidade, fontes, modelos e políticas de dados próprias. Nenhum agregado do MVP pode assumir que só existe `CULTURE`: o campo `domain` é obrigatório e validado por um registry de domínios.

---

## 2. Modelo de propriedade (tenancy)

```
OwnerRef = { ownerType: 'USER' | 'ORGANIZATION', ownerId: ObjectId }
TenantKey = 'user:' + ownerId | 'org:' + ownerId
```

Todo recurso privado carrega `owner` (OwnerRef) e `tenantKey` (string denormalizada para índices e quotas).

Regras:

1. `tenantKey` nunca vem do cliente; é derivado do recurso ou do contexto autenticado.
2. Um recurso de organização só é acessível por `Membership` ativa com papel suficiente.
3. Recurso de usuário não é acessível por organização, exceto quando explicitamente compartilhado (não previsto no MVP).
4. `Membership` é o único caminho para autorização em organização — nunca comparar `organizationId` do body.

---

## 3. Diagrama de relações

```
User 1──1 ProfessionalProfile
  │
  ├──1..n Membership n──1 Organization
  │
  ├──1..n Document (owner=USER)
  │
  └──1..n Project (owner=USER|ORGANIZATION)
              │
              ├──1..n ProjectVersion
              ├──1..n ProjectOpportunity (Proposal) n──1 Opportunity
              │            ├──1..1 Budget ──1..n BudgetItem
              │            └──1..n ProjectVersion (proposal scope)
              ├──1..n ProjectActivity
              └──1..n Document (owner=PROJECT? não; documento pertence a user/org)

Opportunity n──1 OpportunitySource
Opportunity 1──n OpportunityEvidence
Opportunity 1──n EligibilityRule
Opportunity 1──n EditalAnalysis (opcional, via document)
EditalAnalysis n──1 Document
EligibilityAssessment n──1 (Profile × Project × Opportunity)
Subscription n──1 Plan
Notification n──1 User
AuditLog n──1 (actor)
```

> `Document` sempre pertence a um `User` ou `Organization`. A ligação com projeto/análise é por referência (`documentId`), nunca por posse duplicada.

---

## 4. Agregados e invariantes

### 4.1 User

Campos: `id`, `name`, `email` (único, normalizado), `passwordHash`, `emailVerifiedAt`, `status` (`ACTIVE`, `SUSPENDED`, `DELETED`), `acceptedTermsAt`, `acceptedPrivacyAt`, `locale`, `timezone`, `createdAt`, `updatedAt`, `deletedAt`.

Invariantes:

- e-mail único e normalizado (lowercase, trim);
- senha nunca armazenada em claro; hash Argon2id;
- exclusão é soft delete com anonimização após retenção legal;
- usuário `DELETED` não autentica nem aparece em buscas.

### 4.2 Organization

Campos: `id`, `legalName`, `tradeName?`, `cnpj?`, `legalNature?`, `address`, `areas[]`, `responsibleUserId`, `status`, `createdAt`, `updatedAt`.

Invariantes:

- `cnpj` quando presente é único e válido (dígitos verificadores);
- exatamente **um** `OWNER` ativo;
- `OWNER` só pode ser removido após transferência de propriedade;
- organização sem `OWNER` ativo entra em estado inconsistente e gera alerta administrativo.

### 4.3 Membership

Campos: `id`, `organizationId`, `userId`, `role` (`OWNER|ADMIN|EDITOR|VIEWER`), `status` (`INVITED|ACTIVE|SUSPENDED|REMOVED`), `invitedBy`, `invitedAt`, `acceptedAt`.

Invariantes:

- par `(organizationId, userId)` único;
- `INVITED` não autoriza acesso a dados;
- remover o último `OWNER` é proibido;
- mudança de papel gera auditoria.

### 4.4 ProfessionalProfile

Campos: `id`, `userId` (único), `applicantType` (`INDIVIDUAL|MEI|COMPANY|NONPROFIT|COLLECTIVE|PUBLIC`), `areas[]`, `education[]`, `experience[]`, `portfolio[]`, `priorProjects[]`, `skills[]`, `location`, `territorialScope[]`, `summary`, `updatedAt`.

Invariantes:

- um perfil por usuário;
- `applicantType` é dado sensível à elegibilidade: alteração gera novo `profileVersion` (histórico), nunca mutação silenciosa;
- perfil **não** altera versões congeladas (RF-023).

### 4.5 Opportunity

Campos principais: `id`, `title`, `slug` (único), `domain` (`CULTURE|HEALTH`), `description`, `organizer`, `sourceId`, `sourceUrl`, `externalId?`, `publicationDate?`, `openingDate?`, `closingDate?`, `amounts { min?, max?, currency }`, `geo { scope: NATIONAL|STATE|MUNICIPAL|INTERNATIONAL, country, state?, city? }`, `modality?`, `eligibleApplicantTypes[]`, `thematicAreas[]`, `requirements[]` (RuleRef), `requiredDocuments[]`, `evaluationCriteria[]`, `status`, `verificationStatus`, `lastVerifiedAt?`, `dedupeKey`, `fingerprint`, `createdBy`, `updatedBy`, `createdAt`, `updatedAt`.

Invariantes:

- **RB-001:** publicar (`OPEN`/`UPCOMING`/`CLOSED`) exige `sourceId`, `sourceUrl` e ao menos uma evidência;
- `closingDate >= openingDate` quando ambos existem;
- `min <= max` quando ambos existem;
- `status = OPEN` exige `closingDate` no futuro **e** `verificationStatus = VERIFIED` no momento da transição;
- `dedupeKey` único por fonte (`sourceId + externalId`), quando `externalId` existe;
- mudança de `domain` é proibida após criação (RB-014);
- exclusão é `ARCHIVED`, nunca física, quando houver projeto vinculado.

### 4.6 OpportunitySource

Campos: `id`, `name`, `organization`, `baseUrl`, `adapter` (`MANUAL|HTTP|API`), `accessPolicy`, `frequency`, `allowed` (bool), `lastRunAt`, `status`, `failureCount`, `notes`.

Invariante: `allowed = false` impede qualquer coleta automática; apenas curadoria manual.

### 4.7 OpportunityEvidence

Campos: `id`, `opportunityId`, `field`, `documentId?`, `url?`, `page?`, `quote?`, `capturedAt`, `hash`, `capturedBy`.

Invariante: append-only; correção cria nova evidência com `supersedes`.

### 4.8 EditalAnalysis

Campos: `id`, `documentId`, `owner`, `domain`, `status` (`QUEUED|EXTRACTING|ANALYZING|READY_FOR_REVIEW|CONFIRMED|FAILED|PARTIAL`), `extraction { textHash, pageCount, isScanned, engine, version }`, `fields[]` (cada um com `key`, `value`, `evidenceRefs[]`, `confidence`, `validation`), `missingInformation[]`, `reviewedBy?`, `reviewedAt?`, `analysisVersion`, `createdAt`.

Invariantes:

- dado só é "confirmado" após `CONFIRMED` por humano (RB-002);
- `CONFIRMED` é imutável; nova extração cria nova `analysisVersion`;
- `missingInformation` nunca é preenchido por inferência;
- análise sempre referencia `documentId` e `owner`.

### 4.9 EligibilityRule

Campos: `id`, `opportunityId`, `code`, `title`, `category` (`APPLICANT_TYPE|LOCATION|AREA|LEGAL_NATURE|DOCUMENT|QUALIFICATION|EXPERIENCE|OTHER`), `severity` (`BLOCKING|WARNING|INFO`), `evaluator` (`ENUM_SET|RANGE|BOOLEAN|DOCUMENT_PRESENT|TEXT_MANUAL`), `params`, `evidenceRefs[]`, `status` (`DRAFT|CONFIRMED|DISABLED`), `version`, `confirmedBy?`, `confirmedAt?`.

Invariantes:

- regra `DRAFT` não participa de avaliação oficial;
- toda regra confirmada tem evidência;
- regra é versionada; alteração cria nova versão e preserva a anterior.

### 4.10 EligibilityAssessment

Campos: `id`, `opportunityId`, `ruleSetVersion`, `subject { userId, profileVersion, projectId? }`, `outcome` (`ELIGIBLE|ELIGIBLE_WITH_PENDING_ITEMS|INELIGIBLE|INSUFFICIENT_INFORMATION`), `results[]` (`ruleId`, `status`, `profileValue`, `evidenceRef`, `message`), `createdAt`, `createdBy`.

Invariantes:

- avaliação referencia a versão exata do conjunto de regras (reprodutibilidade);
- resultado `INELIGIBLE` exige ao menos uma regra `BLOCKING` reprovada;
- `INSUFFICIENT_INFORMATION` quando falta dado de perfil ou requisito não interpretado;
- nunca representa garantia de seleção (RB-020).

### 4.11 Project

Campos: `id`, `owner`, `domain`, `title`, `summary`, `category`, `location`, `targetAudience`, `estimatedBudget?` (`Money`), `status`, `draft` (`{ sections, interview }` — rascunho editável, não versionado), `sectionPlan?` (configuração do edital congelada no vínculo), `currentVersionId?`, `createdBy`, `createdAt`, `updatedAt`, `archivedAt?`.

Invariantes:

- `domain` imutável após criação (RB-014);
- `status` segue a máquina de estados (§5.2);
- projeto arquivado não aceita novas versões;
- projeto não é excluído fisicamente quando há versão congelada.

### 4.12 ProjectVersion

Campos: `id`, `projectId`, `proposalId?`, `versionNumber`, `status` (`DRAFT|FROZEN|SUPERSEDED`), `content` (estrutura de seções), `contentHash`, `changeSummary`, `authorId`, `createdAt`, `frozenAt?`, `frozenReason?`.

Invariantes:

- `(projectId, proposalId, versionNumber)` único;
- `FROZEN` é imutável (RB-011); qualquer alteração cria nova versão;
- `contentHash` SHA-256 do conteúdo canônico;
- versão congelada referencia as entradas usadas (perfil, edital, prompts) via `provenance`.

### 4.13 ProjectOpportunity (Proposta)

Campos: `id`, `projectId`, `opportunityId`, `owner`, `status` (`DRAFT|IN_DEVELOPMENT|READY_TO_SUBMIT|SUBMITTED|APPROVED|REJECTED|WITHDRAWN`), `currentVersionId?`, `submittedVersionId?`, `submittedAt?`, `budgetId?`, `snapshot` (requisitos do edital no momento do vínculo), `createdAt`.

Invariantes:

- par `(projectId, opportunityId)` único;
- `SUBMITTED` exige `submittedVersionId` congelada;
- mudança de requisito no edital não altera snapshot de proposta já submetida;
- exclusão de projeto com proposta submetida é proibida.

### 4.14 Budget / BudgetItem

`Budget`: `id`, `owner`, `projectId`, `proposalId?`, `currency`, `total` (Decimal128), `status` (`DRAFT|VALIDATED|INVALID`), `limits` (do edital), `version`.

`BudgetItem`: `id`, `budgetId`, `description`, `category`, `quantity` (Decimal128), `unit`, `unitPrice` (Decimal128), `total` (Decimal128), `justification`, `activityId?`, `order`.

Invariantes:

- `total = quantity × unitPrice` calculado no backend;
- `budget.total = Σ items.total` recalculado a cada escrita;
- toda escrita é transacional no agregado `Budget`;
- item sem justificativa é permitido em `DRAFT`, mas bloqueia `VALIDATED` (RF-075).

### 4.15 ProjectActivity

Campos: `id`, `projectId`, `proposalId?`, `stage`, `title`, `description?`, `startDate`, `endDate`, `responsibleId?`, `dependsOn[]`, `status`.

Invariantes:

- `endDate >= startDate`;
- dependências não podem formar ciclo;
- atividade referencia etapa (`stage`) — ver `OPEN-004`.

### 4.16 Document / DocumentVersion

`Document`: `id`, `owner`, `category`, `name`, `mime`, `size`, `hash`, `storageKey`, `scanStatus` (`PENDING|CLEAN|INFECTED|FAILED`), `source` (`UPLOAD|GENERATED`), `linkedTo { type, id }?`, `retentionUntil?`, `createdAt`, `deletedAt?`.

`DocumentVersion`: `id`, `documentId`, `version`, `storageKey`, `hash`, `size`, `createdAt`.

Invariantes:

- download só com autorização e URL assinada;
- hash SHA-256 para integridade;
- `INFECTED` nunca é disponibilizado;
- documento gerado referencia a versão de origem.

### 4.17 Plan / Subscription / UsageRecord

`Plan`: `code`, `name`, `price` (Money), `currency`, `limits` (mapa métrica→limite), `features` (mapa feature→bool), `version`, `active`.

`Subscription`: `id`, `tenantKey`, `planCode`, `status`, `currentPeriodStart`, `currentPeriodEnd`, `trialEndsAt?`, `cancelAt?`, `provider`, `providerCustomerId`, `providerSubscriptionId`, `createdAt`.

`UsageRecord`: `id`, `tenantKey`, `metric`, `period` (`YYYY-MM`), `used`, `reserved`, `updatedAt`.

Invariantes:

- uma assinatura ativa por tenant;
- `providerSubscriptionId` único quando presente;
- limites e features vêm do plano vigente no período;
- overrides administrativos são auditados e temporários.

### 4.18 Notification / SavedSearch

`Notification`: `id`, `userId`, `type`, `title`, `body`, `payload`, `readAt?`, `createdAt`, `channelsSent[]`.

`SavedSearch` (derivada de RF-037): `id`, `owner`, `name`, `filters`, `frequency` (`DAILY|WEEKLY|OFF`), `channels[]`, `lastRunAt?`, `lastNotifiedAt?`.

### 4.19 AuditLog

Campos: `id`, `tenantKey`, `actor { userId, role }`, `action`, `resourceType`, `resourceId`, `before?`, `after?`, `ip`, `userAgent`, `requestId`, `createdAt`.

Invariante: append-only; sem update/delete pela aplicação.

### 4.20 AI

`AiJob`: `id`, `type` (`AI-001..AI-010`), `owner`, `inputRefs[]`, `status`, `attempts`, `resultRef?`, `error?`, `createdAt`.

`AiInteraction`: `id`, `jobId`, `provider`, `model`, `promptId`, `promptVersion`, `inputHash`, `outputHash?`, `tokensIn`, `tokensOut`, `latencyMs`, `costEstimate`, `status`, `createdAt`.

`PromptTemplate`: `id`, `taskId`, `version`, `body`, `outputSchemaVersion`, `status` (`DRAFT|ACTIVE|RETIRED`), `createdAt`, `activatedAt?`.

---

## 5. Máquinas de estado

### 5.1 Oportunidade

**Status editorial (persistido):**

```
DRAFT ──► UNDER_REVIEW ──► OPEN
  │            │             │
  │            ├──► UPCOMING │
  │            ├──► SUSPENDED│
  │            └──► CANCELLED│
  └──► ARCHIVED              │
                             ├──► CLOSED ──► ARCHIVED
                             ├──► SUSPENDED ──► OPEN|CLOSED|CANCELLED
                             └──► CANCELLED ──► ARCHIVED
DISCOVERED ──► UNDER_REVIEW
```

**Regras de transição:**

| De | Para | Guarda |
|:--|:--|:--|
| DRAFT | UNDER_REVIEW | título, domínio, fonte e ao menos 1 evidência |
| UNDER_REVIEW | OPEN | `closingDate > now` **e** `verificationStatus = VERIFIED` |
| UNDER_REVIEW | UPCOMING | `openingDate > now` e verificação válida |
| OPEN | CLOSED | `now >= closingDate` (job) ou decisão editorial com motivo |
| CLOSED | OPEN | nova verificação + `closingDate` futuro + evidência nova |
| qualquer | ARCHIVED | sem projeto vinculado ou com confirmação |

**Status efetivo (derivado, não persistido):**

```
effectiveStatus =
  se status ∈ {OPEN, UPCOMING} e closingDate < now  → CLOSED
  senão → status
```

`confirmationPending = verificationStatus ≠ VERIFIED ou lastVerifiedAt < now - janela` (janela definida em configuração; ver `OPEN-018`).

**Invariante central (RF-038/RB-019):** nenhuma listagem pode exibir `OPEN` quando `effectiveStatus = CLOSED`. A UI mostra "Encerrado — confirmação pendente" quando aplicável, jamais "Aberto".

### 5.2 Projeto

```
IDEA ──► DRAFT ──► IN_DEVELOPMENT ──► UNDER_REVIEW ──► READY_TO_SUBMIT ──► SUBMITTED
                                                              │                 │
                                                              │                 ├──► APPROVED ──► IN_EXECUTION ──► COMPLETED ──► ARCHIVED
                                                              │                 └──► REJECTED ──► IN_DEVELOPMENT
                                                              └──► IN_DEVELOPMENT
```

Guardas:

- `READY_TO_SUBMIT`: validação de requisitos, orçamento e documentos sem bloqueios;
- `SUBMITTED`: existe versão congelada e `submittedAt` registrado;
- `APPROVED`/`REJECTED`: registro manual do usuário (resultado externo);
- `ARCHIVED`: terminal; reabertura cria novo projeto (decisão `OPEN-006`).

**Status do projeto com múltiplas propostas.** O status do projeto reflete a proposta mais avançada, na ordem `APPROVED > SUBMITTED > READY_TO_SUBMIT > IN_DEVELOPMENT > DRAFT`. A fonte de verdade é sempre o status de cada proposta; o campo do projeto é agregado para listagem e dashboard. `IN_EXECUTION`, `COMPLETED` e `ARCHIVED` são manuais.

### 5.3 Proposta (ProjectOpportunity)

```
DRAFT ──► IN_DEVELOPMENT ──► READY_TO_SUBMIT ──► SUBMITTED ──► APPROVED | REJECTED
                                                      └──► WITHDRAWN
```

`SUBMITTED` congela a versão; `WITHDRAWN` não apaga a versão.

### 5.4 Assinatura

```
TRIAL ──► ACTIVE ──► PAST_DUE ──► ACTIVE
             │            ├──► SUSPENDED ──► ACTIVE
             │            └──► CANCELLED ──► EXPIRED
             └──► CANCELLED ──► EXPIRED
```

- `PAST_DUE` mantém acesso de leitura e bloqueia novas gerações caras (configurável).
- `CANCELLED` não apaga projetos (RB-013).
- `EXPIRED` é terminal para o ciclo; reativação cria nova assinatura.

### 5.5 Versão

```
DRAFT ──► FROZEN ──► SUPERSEDED
```

Sem transição de `FROZEN` para `DRAFT`. Correção = nova versão.

---

## 6. Eventos de domínio

| Evento | Emitido por | Consumidores |
|:--|:--|:--|
| `opportunity.published` | Opportunity | notifications, search index, audit |
| `opportunity.closed` | Opportunity (job) | notifications, audit |
| `opportunity.verification_failed` | Opportunity | notifications (admin), audit |
| `edital_analysis.completed` | Edital Analysis | notifications, eligibility |
| `eligibility.assessed` | Eligibility | notifications, audit |
| `project.version_frozen` | Project | audit, export |
| `proposal.submitted` | Proposal | notifications, dashboard, audit |
| `document.scanned` | Document | audit, notifications (se infectado) |
| `subscription.changed` | Billing | entitlements, notifications, audit |
| `quota.exceeded` | Entitlements | notifications, audit |
| `export.ready` | Export | notifications |

Eventos que precisam de garantia usam **outbox**; os demais podem ser in-process.

---

## 7. Versionamento e imutabilidade

| Objeto | Versiona? | Imutável quando | Nova versão criada por |
|:--|:--|:--|:--|
| Proposta | sim | `FROZEN` | edição após congelamento |
| Documento | sim | versão anterior | substituição de arquivo |
| EditalAnalysis | sim | `CONFIRMED` | reprocessamento |
| EligibilityRule | sim | `CONFIRMED` | alteração de requisito |
| PromptTemplate | sim | `ACTIVE` | mudança de prompt |
| Perfil | sim (histórico) | — | alteração relevante |

**Proveniência obrigatória** (RB-010): toda versão congelada registra `provenance { profileVersionId, editalAnalysisVersion, promptIds[], generatedAt, model }`.

---

## 8. Dados derivados e cache

| Dado | Fonte | Estratégia |
|:--|:--|:--|
| `effectiveStatus` | oportunidade + now | calculado em leitura |
| `tenantKey` | owner | denormalizado na escrita |
| quotas do mês | `usage_records` + Redis | Redis atômico; reconciliação periódica |
| contagem de projetos ativos | `projects` | cache invalidado por evento |
| custo de IA do dia | `ai_interactions` | agregação + cache |

---

## 9. Regras de negócio mapeadas

| Regra | Onde vive |
|:--|:--|
| RB-001 fonte verificável | Opportunity (invariante de publicação) |
| RB-002 dado não confirmado | EditalAnalysis + UI (`validationStatus`) |
| RB-003 UTC | camada `shared/dates` + serialização |
| RB-004 Decimal128 | `Money` + schemas |
| RB-005 owner/tenant | todos os agregados privados |
| RB-006 autorização backend | guards |
| RB-007 IDOR | repositórios com TenantContext |
| RB-008 IA não submete | ausência de endpoint/ação; export é manual |
| RB-009 IA não inventa | schema + `missingInformation` |
| RB-010 proveniência | `provenance` em versões + `ai_interactions` |
| RB-011 submetido imutável | `ProjectVersion.FROZEN` |
| RB-012 limites configuráveis | `Plan.limits` |
| RB-013 cancelamento preserva projetos | Subscription (sem cascata) |
| RB-014 domínio imutável | Project (invariante) |
| RB-015 separação cultural/saúde | `domain` + registry |
| RB-016 auditoria | AuditLog |
| RB-017 idempotência financeira | webhooks + outbox |
| RB-018 arquivo não confiável | pipeline de extração e IA |
| RB-019 publicado/aberto/verificado | `status` + `effectiveStatus` + `verificationStatus` |
| RB-020 sem garantia | textos e UI de elegibilidade |
