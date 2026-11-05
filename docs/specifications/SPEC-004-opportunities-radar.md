# SPEC-004 — Oportunidades e Radar

| Campo | Valor |
|:--|:--|
| ID | SPEC-004 |
| Versão | 0.1 |
| Status | Aprovada (2026-11-05) |
| Fase | 1 |
| Dono | Backend |
| Depende de | SPEC-000, SPEC-001, SPEC-002 |
| Requisitos cobertos | RF-030..039, RB-001, RB-002, RB-019 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Manter um catálogo confiável de oportunidades com fonte verificável, busca e filtros úteis, favoritos, alertas e curadoria — nunca exibindo oportunidade encerrada como aberta.

## 2. Fora de escopo

- Coleta automática de portais sem API e sem autorização (conector é Fase 2; MVP aceita curadoria manual assistida).
- Inscrição/submissão em nome do usuário.
- Recomendação personalizada por ML (MVP usa regras simples de match).

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-030 | cadastro estruturado com todos os campos listados |
| RF-031 | busca por título, descrição, organizador e palavras-chave |
| RF-032 | filtros: área, estado, município, abrangência, valor, modalidade, tipo de proponente, prazo, situação |
| RF-033 | estados editoriais definidos |
| RF-034 | evidência de verificação por oportunidade |
| RF-035 | detecção de duplicidade |
| RF-036 | favoritos por usuário |
| RF-037 | alertas por busca salva |
| RF-038 | integridade temporal |
| RF-039 | administração de curadoria |

## 4. Regras e invariantes

- **RB-001:** publicar exige `sourceId`, `sourceUrl` e ≥ 1 evidência.
- **RB-002:** dado importado nasce `verificationStatus = PENDING`; só vira `VERIFIED` com confirmação humana ou conector autorizado.
- **RB-019:** `status` (editorial), `verificationStatus` e `effectiveStatus` (derivado) são distintos.
- `effectiveStatus = CLOSED` quando `closingDate < now` e `status ∈ {OPEN, UPCOMING}`.
- `status = OPEN` exige `closingDate` futuro e `verificationStatus = VERIFIED`.
- `domain` imutável após criação.
- `dedupeKey = sourceId + ':' + externalId` (quando `externalId` existe) é único.
- `fingerprint` = hash normalizado de `title + organizer + closingDate` para sugerir duplicatas; **nunca** mesclar automaticamente.
- Exclusão é `ARCHIVED`; oportunidade com proposta vinculada não é apagada.
- `confirmationPending` é verdadeiro quando `verificationStatus ∈ {PENDING, STALE, FAILED}` ou `lastVerifiedAt` fora da janela.

## 5. Modelo de dados

### `opportunities`

| Campo | Tipo | Nota |
|:--|:--|:--|
| `_id` | ObjectId | |
| `title` | string | 3..240 |
| `slug` | string | único, gerado |
| `domain` | enum | `CULTURE`/`HEALTH` (registry) |
| `description` | string | até 20.000 |
| `organizer` | string | |
| `sourceId` | ObjectId | obrigatório para publicar |
| `sourceUrl` | string | URL http(s) |
| `externalId` | string? | id na fonte |
| `publicationDate` / `openingDate` / `closingDate` | Date? | UTC |
| `amounts` | `{ min?: Decimal128, max?: Decimal128, currency: 'BRL' }` | |
| `geo` | `{ scope, country, state?, city? }` | scope: `NATIONAL / STATE / MUNICIPAL / INTERNATIONAL` |
| `modality` | string? | vocabulário `OPEN-016` |
| `eligibleApplicantTypes` | enum[] | mesma lista de `applicantType` |
| `thematicAreas` | string[] | |
| `requirements` | `{ ruleId, version }[]` | referência a `eligibility_rules` |
| `requiredDocuments` | `{ code, label, mandatory, notes? }[]` | |
| `evaluationCriteria` | `{ criterion, weight?, notes? }[]` | |
| `status` | enum | §6 |
| `verificationStatus` | enum | `PENDING / VERIFIED / STALE / FAILED` |
| `lastVerifiedAt` | Date? | |
| `dedupeKey` | string? | índice único parcial |
| `fingerprint` | string | índice |
| `createdBy` / `updatedBy` | ObjectId | |
| `createdAt` / `updatedAt` | Date | |

### `opportunity_sources`

`{ _id, name, organization, baseUrl, adapter: MANUAL|HTTP|API, accessPolicy, frequency, allowed: boolean, lastRunAt?, status: ACTIVE|PAUSED|ERROR, failureCount, notes }`

### `opportunity_evidence`

`{ _id, opportunityId, field, documentId?, url?, page?, quote?, hash, capturedAt, capturedBy, supersedes? }`

### `opportunity_favorites`

`{ _id, userId, opportunityId, createdAt }` — único por par.

### `saved_searches`

`{ _id, owner, name, filters, frequency: DAILY|WEEKLY|OFF, channels: ['IN_APP','EMAIL'], lastRunAt?, lastNotifiedAt? }`

## 6. Estados

```
DRAFT → UNDER_REVIEW → OPEN | UPCOMING | SUSPENDED | CANCELLED | ARCHIVED
DISCOVERED → UNDER_REVIEW
OPEN → CLOSED | SUSPENDED | CANCELLED | ARCHIVED
UPCOMING → OPEN | CANCELLED | ARCHIVED
CLOSED → ARCHIVED | OPEN (nova verificação + prazo futuro)
SUSPENDED → OPEN | CLOSED | CANCELLED
CANCELLED → ARCHIVED
```

Guardas detalhadas em `DOMAIN_MODEL.md` §5.1.

## 7. Contratos de API

### Leitura (autenticado)

| Método | Rota | Descrição |
|:--|:--|:--|
| GET | `/opportunities` | lista paginada com filtros |
| GET | `/opportunities/search?q=` | busca textual (Atlas Search) |
| GET | `/opportunities/{opportunityId}` | detalhe + evidências + `effectiveStatus` |
| POST | `/opportunities/{opportunityId}/favorite` | idempotente |
| DELETE | `/opportunities/{opportunityId}/favorite` | idempotente |
| GET | `/opportunities/favorites` | lista do usuário |
| GET | `/saved-searches` · POST · PATCH · DELETE | alertas |

### Curadoria (PLATFORM_ADMIN)

| Método | Rota | Descrição |
|:--|:--|:--|
| POST | `/admin/opportunities` | cria DRAFT |
| PATCH | `/admin/opportunities/{id}` | edita |
| POST | `/admin/opportunities/{id}/publish` | valida guardas e publica |
| POST | `/admin/opportunities/{id}/verify` | registra evidência e status |
| POST | `/admin/opportunities/{id}/archive` | arquiva |
| GET | `/admin/opportunities/duplicates` | sugestões por fingerprint |
| POST | `/admin/opportunities/{id}/merge` | merge manual auditado |

### Parâmetros de `/opportunities`

`q`, `area`, `state`, `city`, `scope`, `amountMin`, `amountMax`, `modality`, `applicantType`, `closingWithinDays`, `status` (efetivo), `favorite`, `sort` (`closingDate|publicationDate|relevance`), `cursor`, `limit` (máx. 50).

Resposta:

```json
{
  "items": [{
    "id": "...", "title": "...", "slug": "...", "domain": "CULTURE",
    "organizer": "...", "closingDate": "2026-12-01T23:59:59Z",
    "status": "OPEN", "effectiveStatus": "OPEN", "verificationStatus": "VERIFIED",
    "confirmationPending": false,
    "amounts": { "min": "10000.00", "max": "80000.00", "currency": "BRL" },
    "geo": { "scope": "NATIONAL", "country": "BR" }
  }],
  "nextCursor": "..."
}
```

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 422 | `SOURCE_REQUIRED` | publicar sem fonte/evidência |
| 409 | `OPPORTUNITY_DUPLICATE` | `dedupeKey` já existe |
| 409 | `INVALID_STATUS_TRANSITION` | transição proibida |
| 422 | `INVALID_DATE_RANGE` | `closingDate < openingDate` |
| 422 | `INVALID_AMOUNT_RANGE` | `min > max` |
| 409 | `OPPORTUNITY_HAS_PROPOSALS` | arquivar/merge com propostas |
| 404 | `OPPORTUNITY_NOT_FOUND` | id inexistente ou sem acesso |

## 8. Busca e filtros

- Texto: Atlas Search com analyzers pt-BR; pesos título > organizador > descrição.
- Filtros estruturados aplicados antes do texto quando possível.
- `status` filtra por `effectiveStatus`.
- `closingWithinDays` usa `closingDate` entre agora e agora + N dias.
- Resultado sempre inclui `confirmationPending` e `lastVerifiedAt`.

## 9. Jobs

| Job | Frequência | Comportamento |
|:--|:--|:--|
| `opportunity.temporal-check` | diário | fecha por data, marca `PENDING`, emite evento |
| `opportunity.verification` | por fonte (config) | consulta fonte autorizada; `FAILED` não reabre |
| `opportunity.alerts` | diário/semanal | casa buscas salvas e enfileira notificações |
| `opportunity.index` | sob demanda | reindexa Atlas Search |

## 10. Critérios de aceite

```gherkin
Cenário: encerrada nunca aparece como aberta
  Dado uma oportunidade OPEN com closingDate no passado
  Quando o usuário consulta a lista
  Então o item retorna effectiveStatus CLOSED
  E confirmationPending é true quando não houve nova verificação

Cenário: publicação sem fonte é bloqueada
  Dado um rascunho sem sourceId
  Quando o administrador tenta publicar
  Então a resposta é 422 SOURCE_REQUIRED

Cenário: favorito é isolado por usuário
  Dado que A favoritou uma oportunidade
  Quando B consulta seus favoritos
  Então a oportunidade não aparece

Cenário: duplicidade não é mesclada automaticamente
  Dado duas oportunidades com mesmo fingerprint
  Quando a segunda é criada
  Então é permitida com aviso
  E aparece em /admin/opportunities/duplicates

Cenário: alerta dispara uma vez por período
  Dado uma busca salva diária
  Quando o job roda duas vezes no mesmo dia
  Então apenas uma notificação é criada por oportunidade
```

## 11. Testes

- **Unitários:** derivação de `effectiveStatus`, guardas de transição, fingerprint, normalização de filtros.
- **Integração:** criação/publicação/verificação, unicidade de `dedupeKey`, favoritos, saved searches.
- **E2E:** buscar, filtrar, abrir detalhe, favoritar, criar alerta.
- **Segurança:** curadoria exige PLATFORM_ADMIN; favoritos isolados; nenhum endpoint aceita tenant do cliente.

## 12. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-016 | vocabulário de modalidade | médio | definir com Product |
| OPEN-017 | oportunidades visíveis sem login (SEO) | alto | decidir antes do lançamento |
| OPEN-018 | janela de "verificação válida" (dias) | médio | calibrar com curadoria |
| OPEN-019 | fontes automatizadas permitidas | alto | avaliar termos de uso por fonte |

## 13. Definition of Done específico

- [ ] Nenhuma listagem retorna `OPEN` com prazo vencido (teste dedicado).
- [ ] Publicação sem fonte/evidência impossível por API e por banco.
- [ ] Curadoria auditada.
- [ ] Alertas idempotentes.
- [ ] Índices de busca criados e medidos.

## 14. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
