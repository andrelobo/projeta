# SPEC-008 — Projetos, propostas e versionamento

| Campo | Valor |
|:--|:--|
| ID | SPEC-008 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 |
| Dono | Backend |
| Depende de | SPEC-003, SPEC-004, SPEC-006 |
| Requisitos cobertos | RF-060..069, RB-010, RB-011, RB-014 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Permitir que o usuário transforme uma ideia em projeto estruturado, desenvolva uma proposta para uma oportunidade, versione o conteúdo e congele o que foi submetido — sem que a IA invente dados e sem que edições posteriores alterem o histórico.

## 2. Fora de escopo

- Submissão automática em portais (proibida).
- Orçamento e cronograma detalhados (SPEC-009).
- Colaboração em tempo real.
- Aprovação editorial de conteúdo cultural (responsabilidade do usuário).

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-060 | criação com campos iniciais |
| RF-061 | entrevista estruturada conduzida por IA |
| RF-062 | 19 seções configuráveis conforme edital |
| RF-063 | contexto: usuário, perfil, documentos, edital, modelos |
| RF-064 | IA não inventa; pede complementação |
| RF-065 | edição de todas as seções |
| RF-066 | versão com id, número, data, autor, alterações e conteúdo |
| RF-067 | histórico consultável |
| RF-068 | múltiplas oportunidades por projeto, com histórico próprio |
| RF-069 | versão SUBMITTED imutável |

## 4. Estrutura de conteúdo

### 4.1 Seções canônicas (RF-062)

| # | key | Título |
|--:|:--|:--|
| 1 | `title` | Título |
| 2 | `summary` | Resumo |
| 3 | `presentation` | Apresentação |
| 4 | `justification` | Justificativa |
| 5 | `generalObjective` | Objetivo geral |
| 6 | `specificObjectives` | Objetivos específicos |
| 7 | `targetAudience` | Público-alvo |
| 8 | `methodology` | Metodologia |
| 9 | `activityPlan` | Plano de atividades |
| 10 | `goals` | Metas |
| 11 | `indicators` | Indicadores |
| 12 | `schedule` | Cronograma |
| 13 | `team` | Equipe |
| 14 | `budget` | Orçamento |
| 15 | `communication` | Plano de comunicação |
| 16 | `accessibility` | Acessibilidade |
| 17 | `accessDemocratization` | Democratização de acesso |
| 18 | `sustainability` | Sustentabilidade |
| 19 | `expectedResults` | Resultados esperados |

Cada seção:

```ts
type Section = {
  key: string;
  title: string;
  content: string;              // markdown
  status: 'EMPTY' | 'DRAFT' | 'REVIEWED' | 'APPROVED';
  required: boolean;            // conforme edital
  sources: ('PROFILE' | 'EDITAL' | 'DOCUMENT' | 'USER' | 'AI')[];
  lastGenerationId?: ObjectId;
  updatedAt: Date;
};
```

`sectionPlan` é um snapshot da configuração do edital (ordem, obrigatoriedade, limites de caracteres) gravado na criação da proposta.

### 4.2 Estrutura versionada

`ProjectVersion.content = { sections: Section[], metadata: { title, summary, domain } }` + `contentHash`.

## 5. Modelo de dados

### `projects`

`{ _id, owner, domain, title, summary, category?, location?, targetAudience?, estimatedBudget?, status, draft: { sections: Section[], interview: QA[] }, sectionPlan?, currentVersionId?, createdBy, createdAt, updatedAt, archivedAt? }`

- `draft` é o rascunho editável (não versionado).
- `sectionPlan` congela a configuração do edital no momento do vínculo.

### `project_versions`

`{ _id, projectId, proposalId?, versionNumber, status: DRAFT|FROZEN|SUPERSEDED, content, contentHash, changeSummary, authorId, provenance, createdAt, frozenAt?, frozenReason? }`

### `project_opportunities` (proposta)

`{ _id, projectId, opportunityId, owner, status, currentVersionId?, submittedVersionId?, submittedAt?, snapshot, createdAt }`

`snapshot` inclui requisitos e `sectionPlan` do edital no momento do vínculo.

## 6. Máquina de estados

Projeto e proposta seguem `DOMAIN_MODEL.md` §5.2 e §5.3.

**Projeto com múltiplas propostas:** o status do projeto reflete a proposta mais avançada (`APPROVED > SUBMITTED > READY_TO_SUBMIT > IN_DEVELOPMENT > DRAFT`). O status individual da proposta é a fonte de verdade; o do projeto é agregado para listagem/dashboard. `IN_EXECUTION`, `COMPLETED` e `ARCHIVED` são decisões manuais.

Guardas adicionais:

- `READY_TO_SUBMIT`: todas as seções obrigatórias com `status ∈ {REVIEWED, APPROVED}`; elegibilidade não `INELIGIBLE`; sem pendência bloqueante de documentos.
- `SUBMITTED`: `submittedVersionId` existe e está `FROZEN`.
- `ARCHIVED`: projeto com proposta `SUBMITTED` só arquiva; não exclui.

## 7. Contratos de API

| Método | Rota | Permissão | Request | Response |
|:--|:--|:--|:--|:--|
| POST | `/projects` | autenticado | `{ title, domain, summary?, category?, location?, targetAudience?, organizationId? }` | `201` projeto |
| GET | `/projects` | autenticado | filtros + paginação | lista |
| GET | `/projects/{projectId}` | dono/EDITOR+ | — | projeto + draft |
| PATCH | `/projects/{projectId}` | dono/EDITOR+ | campos e seções | `200` |
| DELETE | `/projects/{projectId}` | dono | — | `204` (soft) |
| POST | `/projects/{projectId}/interview` | dono/EDITOR+ | `{ answers? }` | próxima pergunta (AI-005) |
| POST | `/projects/{projectId}/generate-section` | dono/EDITOR+ + quota | `{ sectionKey, instructions?, tone? }` | `202 { jobId }` |
| POST | `/projects/{projectId}/review` | dono/EDITOR+ + quota | `{ scope? }` | `202 { jobId }` (AI-007/008/009) |
| POST | `/projects/{projectId}/versions` | dono/EDITOR+ | `{ changeSummary }` | `201` versão `DRAFT` |
| GET | `/projects/{projectId}/versions` | dono/VIEWER+ | paginação | histórico |
| GET | `/projects/{projectId}/versions/{versionId}` | dono/VIEWER+ | — | conteúdo |
| POST | `/projects/{projectId}/versions/{versionId}/freeze` | dono | `{ reason }` | `200` |
| POST | `/projects/{projectId}/proposals` | dono/EDITOR+ | `{ opportunityId }` | `201` proposta |
| GET | `/projects/{projectId}/proposals` | dono/VIEWER+ | — | propostas |
| POST | `/projects/{projectId}/proposals/{proposalId}/submit` | dono | `{ versionId, submittedAt, notes? }` | `200` |
| POST | `/projects/{projectId}/proposals/{proposalId}/withdraw` | dono | `{ reason }` | `200` |

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 409 | `PROJECT_DOMAIN_IMMUTABLE` | tentar mudar domínio |
| 409 | `VERSION_FROZEN` | editar versão congelada |
| 409 | `PROJECT_HAS_SUBMITTED_PROPOSAL` | excluir projeto submetido |
| 422 | `REQUIRED_SECTIONS_MISSING` | marcar pronto com seção obrigatória vazia |
| 422 | `OPPORTUNITY_CLOSED` | vincular a oportunidade encerrada |
| 409 | `PROPOSAL_ALREADY_EXISTS` | mesmo projeto + oportunidade |
| 429 | `QUOTA_EXCEEDED` | limite de gerações |
| 502 | `AI_INVALID_OUTPUT` | geração falhou |

## 8. Geração e proveniência

1. `generate-section` cria `ai_job` e enfileira.
2. Worker monta contexto: seção atual, demais seções, perfil, edital confirmado, documentos autorizados, tom/instruções.
3. Saída validada: `{ content, usedSources[], missingInformation[], assumptions[] }`.
4. Conteúdo é salvo no **draft** com `status: DRAFT` e `lastGenerationId`.
5. `missingInformation` vira pendência visível; nada é inventado.
6. `provenance` da versão registra `promptId`, `promptVersion`, `model`, `inputHash`, `jobId`.

Regra: geração nunca sobrescreve seção `APPROVED` sem confirmação do usuário.

## 9. Entrevista (RF-061)

- AI-005 conduz perguntas por lacuna (público, local, duração, equipe, orçamento, resultados).
- Respostas são persistidas em `draft.interview`.
- A entrevista nunca bloqueia a edição manual.
- Perguntas com resposta já existente no perfil não são repetidas.

## 10. Critérios de aceite

```gherkin
Cenário: versão congelada é imutável
  Dado uma versão FROZEN
  Quando o usuário tenta editar seu conteúdo
  Então a resposta é 409 VERSION_FROZEN
  E o conteúdo permanece idêntico

Cenário: submissão exige versão congelada
  Dado uma proposta sem versão congelada
  Quando o usuário registra submissão
  Então a resposta é 422
  E nenhuma submissão é registrada

Cenário: IA não inventa participante
  Dado um projeto sem equipe informada
  Quando a seção "Equipe" é gerada
  Então o conteúdo não cita nomes
  E missingInformation inclui "equipe"

Cenário: duas propostas com históricos distintos
  Dado um projeto vinculado a duas oportunidades
  Quando cada proposta recebe uma versão
  Então cada histórico contém apenas suas versões

Cenário: domínio imutável
  Dado um projeto CULTURE
  Quando se tenta alterar para HEALTH
  Então a resposta é 409 PROJECT_DOMAIN_IMMUTABLE
```

## 11. Testes

- **Unitários:** hashing de conteúdo, guardas de transição, validação de seções obrigatórias, cálculo de `sectionPlan`.
- **Integração:** criação, geração com provider fake, versão, congelamento, submissão, duas propostas.
- **E2E:** criar projeto → entrevista → gerar seções → revisar → versionar → congelar → exportar.
- **Segurança:** acesso de outro tenant; edição por VIEWER; geração sem quota.

## 12. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-030 | autosave vs versão explícita | médio | decidir com UX |
| OPEN-031 | limite de caracteres por seção | baixo | extrair do edital |
| OPEN-032 | modelos de seção por oportunidade | médio | Fase 2 |
| OPEN-006 | reabertura de projeto arquivado | baixo | definir regra |

## 13. Definition of Done específico

- [ ] Versão congelada comprovadamente imutável (teste).
- [ ] Proveniência registrada em toda geração.
- [ ] `missingInformation` visível no frontend.
- [ ] Duas propostas com históricos independentes testadas.
- [ ] Exportação usa `versionId` explícito.

## 14. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
