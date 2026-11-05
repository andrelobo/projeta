# SPEC-009 — Orçamento e cronograma

| Campo | Valor |
|:--|:--|
| ID | SPEC-009 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 2 |
| Dono | Backend |
| Depende de | SPEC-008, SPEC-010 |
| Requisitos cobertos | RF-070..075, RF-080..083, RB-004 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Permitir construir, validar e exportar orçamento e cronograma coerentes com o edital, com cálculo exato no backend e alertas de inconsistência antes da submissão.

## 2. Fora de escopo

- Execução financeira do projeto aprovado.
- Prestação de contas avançada.
- Integração contábil.
- Aprovação automática de valores.

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-070 | categorias configuráveis |
| RF-071 | itens com descrição, categoria, quantidade, unidade, valor unitário, total, justificativa e etapa |
| RF-072 | total = quantidade × valor unitário; somatório no backend com decimal |
| RF-073 | comparação com limites do edital |
| RF-074 | exportação XLSX e PDF |
| RF-075 | validação de ausências, totais, categorias, limites e justificativas |
| RF-080 | atividades com responsáveis, datas e dependências |
| RF-081 | organização por etapas |
| RF-082 | validação de datas e prazos do edital |
| RF-083 | visualização em lista e calendário |

## 4. Orçamento — regras

- `quantity`, `unitPrice` e `total` são `Decimal128`/string decimal; nunca `number`.
- `item.total = quantity × unitPrice`, arredondado para 2 casas (half-up) **no backend**.
- `budget.total = Σ item.total`, recalculado a cada escrita.
- Moeda única por orçamento; padrão `BRL`.
- Item sem justificativa: permitido em `DRAFT`; bloqueia `VALIDATED`.
- Categoria fora da lista do edital: erro de validação.
- Limites: `min`, `max` e limite por categoria (quando o edital definir).
- Etapa (`stage`) deve existir no cronograma ou na lista de etapas do projeto.
- Alterar item de orçamento `VALIDATED` volta status para `DRAFT`.

### Validações (RF-075)

| Código | Condição | Severidade |
|:--|:--|:--|
| `ITEM_WITHOUT_JUSTIFICATION` | justificativa vazia | erro |
| `ITEM_WITHOUT_CATEGORY` | categoria ausente | erro |
| `INVALID_QUANTITY` | ≤ 0 | erro |
| `INVALID_UNIT_PRICE` | < 0 | erro |
| `CATEGORY_NOT_ALLOWED` | fora da lista do edital | erro |
| `BUDGET_BELOW_MIN` | total < min do edital | aviso |
| `BUDGET_ABOVE_MAX` | total > max | erro |
| `CATEGORY_LIMIT_EXCEEDED` | acima do limite da categoria | erro |
| `ITEM_WITHOUT_ACTIVITY` | etapa obrigatória e ausente | aviso |

## 5. Cronograma — regras

- `endDate >= startDate`.
- Atividade fora da janela do edital → aviso/erro conforme configuração.
- Dependência inexistente → erro.
- Ciclo de dependências → erro (`SCHEDULE_CYCLE`).
- Etapas ordenadas; atividade pertence a uma etapa.
- Responsável deve ser membro do tenant (usuário ou colaborador cadastrado).
- Datas em UTC; exibição no fuso do usuário.

## 6. Modelo de dados

### `budgets`

`{ _id, owner, projectId, proposalId?, currency, total, status: DRAFT|VALIDATED|INVALID, limits: { min?, max?, categories? }, version, createdAt, updatedAt }`

### `budget_items`

`{ _id, budgetId, description, category, quantity, unit, unitPrice, total, justification?, activityId?, order }`

### `project_activities`

`{ _id, projectId, proposalId?, stage, title, description?, startDate, endDate, responsibleId?, dependsOn: ObjectId[], status: PLANNED|IN_PROGRESS|DONE, order }`

Índices: `{ budgetId: 1, order: 1 }`, `{ projectId: 1, startDate: 1 }`, `{ "dependsOn": 1 }`.

## 7. Contratos de API

| Método | Rota | Permissão | Request | Response |
|:--|:--|:--|:--|:--|
| POST | `/projects/{projectId}/budgets` | EDITOR+ | `{ proposalId?, currency }` | `201` |
| GET | `/projects/{projectId}/budgets` | VIEWER+ | — | lista |
| PATCH | `/budgets/{budgetId}` | EDITOR+ | `{ limits?, status? }` | `200` |
| POST | `/budgets/{budgetId}/items` | EDITOR+ | item | `201` |
| PATCH | `/budgets/{budgetId}/items/{itemId}` | EDITOR+ | parcial | `200` |
| DELETE | `/budgets/{budgetId}/items/{itemId}` | EDITOR+ | — | `204` |
| POST | `/budgets/{budgetId}/validate` | EDITOR+ | — | `200 { status, issues[] }` |
| GET | `/budgets/{budgetId}/export?format=xlsx / pdf` | EDITOR+ | — | `202 { jobId }` |
| POST | `/projects/{projectId}/activities` | EDITOR+ | atividade | `201` |
| PATCH | `/activities/{activityId}` | EDITOR+ | parcial | `200` |
| DELETE | `/activities/{activityId}` | EDITOR+ | — | `204` |
| GET | `/projects/{projectId}/schedule` | VIEWER+ | `from?`, `to?` | atividades + validação |

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 422 | `BUDGET_VALIDATION_FAILED` | validação com erros |
| 422 | `INVALID_AMOUNT` | valor inválido/precisão |
| 409 | `BUDGET_VALIDATED_IMMUTABLE` | editar item validado sem reabrir |
| 422 | `SCHEDULE_CYCLE` | dependência circular |
| 422 | `ACTIVITY_OUT_OF_WINDOW` | fora do prazo do edital |
| 409 | `SCHEDULE_LOCKED` | cronograma de proposta submetida |

## 8. Exportação

- XLSX: uma aba por orçamento, com subtotais por categoria e total.
- PDF: layout legível, com cabeçalho do projeto/oportunidade e data.
- Exportação referencia a versão congelada quando existir; nunca "última alteração".
- Arquivo gerado vira `Document` com `source = GENERATED` e `linkedTo = budget`.

## 9. Critérios de aceite

```gherkin
Cenário: total do item é calculado no backend
  Dado quantidade 3 e valor unitário 10,10
  Quando o item é salvo
  Então total é "30.30"
  E enviar total divergente no request não altera o valor calculado

Cenário: limite do edital é respeitado
  Dado um edital com máximo de R$ 50.000
  Quando o orçamento soma R$ 50.001
  Então validate retorna BUDGET_ABOVE_MAX
  E status é INVALID

Cenário: ciclo de dependências é bloqueado
  Dado A depende de B e B depende de A
  Quando a segunda dependência é salva
  Então a resposta é 422 SCHEDULE_CYCLE

Cenário: item sem justificativa bloqueia validação
  Dado um item sem justificativa
  Quando o usuário valida o orçamento
  Então o status permanece DRAFT
  E o problema é listado
```

## 10. Testes

- **Unitários:** arredondamento decimal, soma, avaliadores de limite, detecção de ciclo.
- **Integração:** CRUD de itens, validação, exportação, atividades e dependências.
- **E2E:** montar orçamento, validar, corrigir, exportar XLSX/PDF.
- **Segurança:** orçamento de outro tenant; exportação sem permissão.

## 11. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-034 | arredondamento (half-up vs banker's) | baixo | definir e testar |
| OPEN-035 | limites por categoria vêm do edital? | médio | depende da análise |
| OPEN-036 | cronograma por etapas do edital | médio | mapear em SPEC-005 |

## 12. Definition of Done específico

- [ ] Nenhum cálculo financeiro em `number`.
- [ ] Testes de arredondamento e limite.
- [ ] Exportação reproduzível por versão.
- [ ] Validação bloqueia submissão inválida.
- [ ] Ciclo de dependência impossível.

## 13. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
