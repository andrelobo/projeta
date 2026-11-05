# SPEC-007 — Motor de elegibilidade

| Campo | Valor |
|:--|:--|
| ID | SPEC-007 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 |
| Dono | Backend |
| Depende de | SPEC-003, SPEC-004, SPEC-005 |
| Requisitos cobertos | RF-050..053, RB-020 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Dizer, com justificativa rastreável, se o perfil (e opcionalmente o projeto) atende aos requisitos de uma oportunidade — sem prometer aprovação e sem depender de julgamento probabilístico para regras objetivas.

## 2. Fora de escopo

- Prever chance de seleção.
- Avaliar mérito artístico.
- Substituir análise jurídica de edital.
- Elegibilidade do domínio HEALTH (Fase 3).

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-050 | compara tipo de proponente, localização, área, formação, natureza jurídica, documentação e requisitos específicos |
| RF-051 | quatro classificações de resultado |
| RF-052 | justificativa por requisito, com evidência do edital e dado do perfil |
| RF-053 | regras objetivas avaliadas de forma determinística; IA apenas auxilia interpretação |

## 4. Modelo de regra

```ts
type EligibilityRule = {
  id: ObjectId;
  opportunityId: ObjectId;
  code: string;                    // estável, ex.: "APPLICANT_TYPE"
  title: string;
  category: 'APPLICANT_TYPE' | 'LOCATION' | 'AREA' | 'LEGAL_NATURE' |
            'DOCUMENT' | 'QUALIFICATION' | 'EXPERIENCE' | 'OTHER';
  severity: 'BLOCKING' | 'WARNING' | 'INFO';
  evaluator: 'ENUM_SET' | 'RANGE' | 'BOOLEAN' | 'DOCUMENT_PRESENT' |
             'TEXT_MANUAL' | 'EXPERIENCE_MIN';
  params: Record<string, unknown>;
  evidence: { page: number; quote: string; documentId: ObjectId }[];
  status: 'DRAFT' | 'CONFIRMED' | 'DISABLED';
  version: number;
  confirmedBy?: ObjectId;
  confirmedAt?: Date;
};
```

### 4.1 Avaliadores

| Evaluator | Entrada | Regra |
|:--|:--|:--|
| `ENUM_SET` | `profile.applicantType` | pertence a `params.allowed` |
| `LOCATION` | `profile.location` / `org.address` | atende `params.scope` e UF/município |
| `AREA` | `profile.areas` | interseção com `params.requiredAny` |
| `LEGAL_NATURE` | `org.legalNature` | pertence a `params.allowed` |
| `DOCUMENT_PRESENT` | documentos do perfil/org | existe documento com `code` exigido |
| `EXPERIENCE_MIN` | `profile.experience` | anos/projetos ≥ `params.min` |
| `RANGE` | valor numérico | dentro de `params.min/max` |
| `BOOLEAN` | campo booleano | igual a `params.expected` |
| `TEXT_MANUAL` | — | exige decisão humana (`UNKNOWN` até revisão) |

Regra `TEXT_MANUAL` **nunca** é resolvida por IA em produção: a IA pode sugerir interpretação (AI-004), mas o resultado fica `PENDING_REVIEW`.

## 5. Avaliação

Para cada regra `CONFIRMED`:

1. obter valor do perfil/organização/documentos;
2. se dado ausente → `UNKNOWN` com `missingProfileField`;
3. aplicar avaliador determinístico;
4. produzir `PASS`, `FAIL` ou `UNKNOWN` + justificativa.

Resultado agregado (precedência):

| Condição | Outcome |
|:--|:--|
| alguma `BLOCKING` = `FAIL` | `INELIGIBLE` |
| nenhuma `BLOCKING` falhou, mas alguma `BLOCKING` = `UNKNOWN` | `INSUFFICIENT_INFORMATION` |
| nenhuma falha bloqueante, mas há `WARNING` = `FAIL`/`UNKNOWN` ou documentos pendentes | `ELIGIBLE_WITH_PENDING_ITEMS` |
| todas `PASS` (ou apenas `INFO`) | `ELIGIBLE` |

A avaliação é **reprodutível**: grava `ruleSetVersion`, `profileVersionHash`, `opportunityVersion` e resultado por regra.

## 6. Modelo de dados

### `eligibility_rules`

Índices: `{ opportunityId: 1, status: 1, code: 1 }`, `{ opportunityId: 1, version: -1 }`.

### `eligibility_assessments`

| Campo | Tipo |
|:--|:--|
| `_id` | ObjectId |
| `opportunityId` | ObjectId |
| `ruleSetVersion` | string (hash do conjunto confirmado) |
| `subject` | `{ userId, profileVersionHash, projectId?, organizationId? }` |
| `outcome` | enum |
| `results` | `{ ruleId, code, status, profileValue, evidenceRef, message }[]` |
| `createdBy` / `createdAt` | ObjectId / Date |

Índices: `{ "subject.userId": 1, opportunityId: 1, createdAt: -1 }`.

## 7. Contratos de API

| Método | Rota | Permissão | Request | Response |
|:--|:--|:--|:--|:--|
| POST | `/eligibility/assessments` | autenticado | `{ opportunityId, projectId?, organizationId? }` | `201` avaliação |
| GET | `/eligibility/assessments/{assessmentId}` | dono | — | avaliação |
| GET | `/eligibility/assessments?opportunityId=` | dono | — | histórico |
| POST | `/admin/opportunities/{id}/rules` | PLATFORM_ADMIN | regra | `201` `DRAFT` |
| PATCH | `/admin/rules/{ruleId}` | PLATFORM_ADMIN | parcial | `200` (nova versão) |
| POST | `/admin/rules/{ruleId}/confirm` | PLATFORM_ADMIN | — | `200` `CONFIRMED` |

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 422 | `PROFILE_INCOMPLETE` | falta dado obrigatório para regra bloqueante |
| 422 | `NO_CONFIRMED_RULES` | oportunidade sem regras confirmadas |
| 409 | `ASSESSMENT_STALE` | regras mudaram desde a avaliação |
| 403 | `ORG_REQUIRED` | regra exige organização e não há |
| 404 | `OPPORTUNITY_NOT_FOUND` | id inválido/sem acesso |

## 8. Interface e linguagem

- Exibir sempre: requisito, dado do perfil, trecho do edital, resultado e pendência.
- Nunca exibir "aprovado", "garantido" ou probabilidade (RB-020).
- Exibir aviso: "elegibilidade não garante seleção; a decisão é do financiador".
- Pendências com link direto para completar perfil/documento.

## 9. Critérios de aceite

```gherkin
Cenário: regra bloqueante reprovada
  Dado que o edital exige MEI e o perfil é INDIVIDUAL
  Quando a avaliação roda
  Então o outcome é INELIGIBLE
  E a justificativa mostra perfil, trecho do edital e regra

Cenário: dado faltante não vira reprovação
  Dado que o edital exige documento X e o perfil não o possui
  Quando a avaliação roda
  Então o resultado da regra é UNKNOWN
  E o outcome é INSUFFICIENT_INFORMATION ou ELIGIBLE_WITH_PENDING_ITEMS

Cenário: reprodutibilidade
  Dado uma avaliação concluída
  Quando o mesmo conjunto de regras e perfil é reavaliado
  Então o outcome e os resultados são idênticos

Cenário: IA não decide
  Dado uma regra TEXT_MANUAL
  Quando a avaliação roda
  Então o status é UNKNOWN
  E nenhuma chamada de IA altera o outcome
```

## 10. Testes

- **Unitários:** cada avaliador, precedência de outcome, hash do rule set.
- **Integração:** avaliação com perfil/organização/documentos reais; regra draft ignorada.
- **E2E:** analisar edital → revisar regras → avaliar → abrir pendências.
- **Segurança:** avaliação de outro tenant; regra alterada após avaliação (stale).
- **Propriedade (property-based):** qualquer combinação de regras produz outcome válido e determinístico.

## 11. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-027 | quem confirma regras (curador vs usuário) | alto | definir com Product |
| OPEN-028 | oportunidades sem regras estruturadas | alto | fallback para TEXT_MANUAL |
| OPEN-029 | requisito de CNAE/atividade | médio | depende de dados da organização |

## 12. Definition of Done específico

- [ ] Nenhum caminho de IA altera outcome.
- [ ] Avaliação reproduzível (mesmo input → mesmo output).
- [ ] Todas as regras com evidência.
- [ ] UI com aviso de não garantia.
- [ ] Teste de stale funcionando.

## 13. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
