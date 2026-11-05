# SPEC-003 — Perfil profissional

| Campo | Valor |
|:--|:--|
| ID | SPEC-003 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 |
| Dono | Backend |
| Depende de | SPEC-000, SPEC-001, SPEC-010 |
| Requisitos cobertos | RF-020, RF-021, RF-022, RF-023, RB-003 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Concentrar os dados profissionais que alimentam elegibilidade e geração de propostas, mantendo histórico e sem alterar versões já congeladas.

## 2. Fora de escopo

- Perfil de organização (dados institucionais ficam em SPEC-002).
- Verificação/validação de currículo por terceiros.
- Upload e curadoria de portfólio público (não há perfil público no MVP).

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-020 | cadastro de áreas, formação, experiência, portfólio, projetos anteriores, habilidades, localização, abrangência e tipo de proponente |
| RF-021 | associação de documentos ao perfil |
| RF-022 | reutilização em projetos somente com confirmação do usuário |
| RF-023 | alteração de perfil não modifica versões congeladas |
| RB-003 | datas em UTC |

## 4. Regras e invariantes

- Um perfil por usuário (`userId` único).
- `applicantType` é obrigatório para avaliação de elegibilidade; valores: `INDIVIDUAL`, `MEI`, `COMPANY`, `NONPROFIT`, `COLLECTIVE`, `PUBLIC`.
- `location` exige ao menos `country`, `state`, `city` quando `country = BR`.
- `territorialScope[]` usa o mesmo vocabulário de `geo` das oportunidades (nacional, estadual, municipal, internacional).
- Alteração de `applicantType`, `location` ou `areas` registra entrada em `history[]` (quem, quando, antes/depois resumido).
- Documentos associados pertencem ao usuário (owner `USER`), nunca à organização.
- Perfil não é copiado para dentro da versão congelada: a versão guarda `provenance.profileVersionId` e o snapshot do contexto usado.

## 5. Modelo de dados

### `professional_profiles`

| Campo | Tipo | Regra |
|:--|:--|:--|
| `_id` | ObjectId | |
| `userId` | ObjectId | único |
| `applicantType` | enum | obrigatório |
| `headline` | string? | até 160 |
| `summary` | string? | até 2.000 |
| `areas` | string[] | vocabulário controlado + livre |
| `education` | `{ institution, course, degree?, startYear?, endYear? }[]` | |
| `experience` | `{ organization, role, startDate?, endDate?, description? }[]` | |
| `portfolio` | `{ label, url }[]` | URL http(s) |
| `priorProjects` | `{ title, year?, role?, summary? }[]` | |
| `skills` | string[] | |
| `location` | `{ country, state?, city?, postalCode? }` | |
| `territorialScope` | string[] | |
| `history` | `{ at, field, beforeHash, afterHash, actorId }[]` | últimos 50 |
| `createdAt` / `updatedAt` | Date | |

### Documentos

Reutiliza `documents` (SPEC-010) com `linkedTo = { type: 'PROFILE', id: profileId }` e `category` pessoal/currículo/portfólio.

## 6. Contratos de API

| Método | Rota | Permissão | Request | Response |
|:--|:--|:--|:--|:--|
| GET | `/profiles/me` | autenticado | — | perfil |
| PUT | `/profiles/me` | autenticado | perfil completo | `200` |
| PATCH | `/profiles/me` | autenticado | parcial | `200` |
| GET | `/profiles/me/completeness` | autenticado | — | `{ score, missing[] }` |
| GET | `/profiles/me/documents` | autenticado | — | documentos do perfil |
| POST | `/profiles/me/documents` | autenticado | `{ documentId }` | `201` vínculo |

> `PUT` é o caminho principal: substitui o documento inteiro, evitando merge ambíguo de listas.

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 422 | `PROFILE_INCOMPLETE` | campo obrigatório ausente em operação que exige |
| 422 | `INVALID_LOCATION` | cidade/estado inconsistentes |
| 409 | `DOCUMENT_ALREADY_LINKED` | vínculo duplicado |

## 7. Reutilização em projetos (RF-022)

1. A geração de seção recebe o perfil como contexto.
2. Campos preenchidos a partir do perfil vêm marcados com `source: "PROFILE"` e `confirmed: false`.
3. A UI destaca e pede confirmação.
4. Somente após confirmação o conteúdo é salvo na versão em rascunho.
5. Versões congeladas não são tocadas (RF-023).

## 8. Critérios de aceite

```gherkin
Cenário: perfil incompleto bloqueia elegibilidade
  Dado um perfil sem applicantType
  Quando o usuário solicita avaliação de elegibilidade
  Então a resposta é 422 PROFILE_INCOMPLETE
  E a lista de campos ausentes é retornada

Cenário: alteração de perfil não altera versão congelada
  Dado um projeto com versão FROZEN
  Quando o usuário altera suas áreas de atuação
  Então a versão congelada permanece idêntica
  E o hash de conteúdo não muda

Cenário: documento de perfil isolado por usuário
  Dado um documento vinculado ao perfil de A
  Quando B tenta acessá-lo
  Então a resposta é 404
```

## 9. Testes

- **Unitários:** validação de localização, cálculo de completude, registro de histórico.
- **Integração:** PUT/PATCH, vínculo de documentos, isolamento por usuário.
- **E2E:** preencher perfil, anexar currículo, ver completude.
- **Segurança:** acesso a perfil/documento de outro usuário.

## 10. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-014 | vocabulário de áreas (controlado vs livre) | médio | definir com Product; impacta busca e elegibilidade |
| OPEN-015 | perfis múltiplos por usuário (pessoa física + PJ) | alto | decidir antes de STUDIO |

## 11. Definition of Done específico

- [ ] Completude e campos ausentes testados.
- [ ] Histórico de alteração registrado.
- [ ] Vínculo de documento com autorização validada.
- [ ] Nenhum dado de perfil em log.

## 12. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
