# SPEC-006 — Plataforma de IA e guardrails

| Campo | Valor |
|:--|:--|
| ID | SPEC-006 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 |
| Dono | Backend / System Architect |
| Depende de | SPEC-000 |
| Requisitos cobertos | AI-001..AI-010, §7 do PRD, RB-009, RB-010, RB-018, RNF-021 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Oferecer uma camada única, auditável e econômica para todas as operações de IA, garantindo: saída estruturada, evidência, ausência de invenção, isolamento por tenant, proteção contra prompt injection e controle de custo.

## 2. Fora de escopo

- Treinamento ou fine-tuning de modelos.
- Agentes autônomos com ferramentas.
- Geração de imagem/vídeo.
- Substituição de regra determinística por julgamento do modelo.

## 3. Arquitetura da camada

```
src/ai/ (dentro de projeta-api)
├── providers/           # adapters (OpenAI, Anthropic, DeepSeek, ...)
├── registry/            # modelos, preços, capacidades
├── prompts/             # templates versionados
├── tasks/               # AI-001..AI-010 (schema, prompt, limites)
├── rag/                 # chunking, retrieval, filtros de tenant
├── guardrails/          # injection, PII, limites, validação
├── cost/                # ledger e orçamento
└── index.ts             # runTask()
```

Interface de provedor:

```ts
type CompletionRequest = {
  model: string;
  system: string;
  messages: { role: 'user' | 'assistant'; content: string }[];
  temperature: number;
  maxOutputTokens: number;
  responseSchema?: JSONSchema;
  timeoutMs: number;
  metadata: { taskId: string; tenantKey: string; requestId: string };
};

type CompletionResponse = {
  text: string;
  usage: { inputTokens: number; outputTokens: number };
  finishReason: 'stop' | 'length' | 'content_filter' | 'error';
  providerRequestId?: string;
};

interface AiProvider {
  id: string;
  supports(model: string): boolean;
  complete(req: CompletionRequest): Promise<CompletionResponse>;
}
```

## 4. Catálogo de tarefas

| Task | Uso | Entrada | Saída (schema) | Tokens saída (máx.) |
|:--|:--|:--|:--|--:|
| AI-001 | identificar edital | texto + metadados | `EditalIdentification` | 1.500 |
| AI-002 | extrair requisitos | texto + chunks | `RequirementsExtraction` | 2.500 |
| AI-003 | explicar regras | requisito + perfil | `RuleExplanation` | 800 |
| AI-004 | apoiar compatibilidade | requisitos + perfil | `EligibilityNarrative` | 1.200 |
| AI-005 | entrevista de projeto | estado do projeto | `InterviewQuestion` | 600 |
| AI-006 | elaborar seção | contexto montado | `SectionDraft` | 3.000 |
| AI-007 | revisar proposta | versão + edital | `ReviewFindings` | 2.000 |
| AI-008 | inconsistências | versão + orçamento | `InconsistencyList` | 1.500 |
| AI-009 | sugestões | versão + critérios | `Suggestions` | 1.500 |
| AI-010 | justificativa orçamentária | item + contexto | `BudgetJustification` | 800 |

Cada tarefa declara `promptId`, `schemaVersion`, `temperature`, `modelClass` (`SMALL|STANDARD|DEEP`) e `cacheable`.

## 5. Prompts

- `PromptTemplate { id, taskId, version, body, outputSchemaVersion, status, createdAt, activatedAt }`.
- Status: `DRAFT → ACTIVE → RETIRED`.
- Uma única versão `ACTIVE` por `taskId`.
- Ativação exige aprovação registrada (quem, quando, motivo).
- Corpo do prompt separa **instruções** de **dados**: dados sempre em bloco delimitado.
- Mudança de prompt nunca altera `ai_interactions` históricas (RB-010).

## 6. Context assembly e RAG

Ordem de montagem:

1. instruções do sistema (fixas, versionadas);
2. tarefa e formato de saída;
3. perfil/projeto (somente campos necessários);
4. trechos recuperados (RAG), cada um com fonte;
5. pergunta/comando do usuário;
6. bloco de dados externos delimitado.

Regras:

- Orçamento de tokens de entrada por tarefa; excedeu → sumarização ou recusa explícita.
- RAG filtra **sempre** por `tenantKey` e `domain`; jamais recupera chunk de outro tenant.
- Cada chunk: `{ documentId, page, section, hash, text, domain, tenantKey }`.
- `topK` e limiar de similaridade configuráveis por tarefa.
- Sem similaridade suficiente → `INSUFFICIENT_CONTEXT` (não inventar).

## 7. Validação de saída

1. Parse JSON (tolerante a cercas de código).
2. Validação Zod; falhou → tentativa de reparo (1x) com erro estruturado.
3. Falhou de novo → `AI_INVALID_OUTPUT`, job falha, nada é persistido como sucesso.
4. Campos desconhecidos são descartados e registrados em `warnings`.
5. `missingInformation[]` é obrigatório nos schemas que produzem conteúdo factual.

## 8. Guardrails

- **Prompt injection:** conteúdo externo em `<documento>`, instrução de que é dado; nenhuma ferramenta disponível; detecção de padrões suspeitos registrada.
- **PII:** redação de CPF/e-mail/telefone em logs; não enviar PII desnecessária ao provedor.
- **Cross-tenant:** filtro obrigatório no retriever e no assembler; teste dedicado.
- **Limites:** `maxOutputTokens`, timeout, número de retries.
- **Conteúdo:** recusa de instruções para burlar regras do produto.
- **Auditoria:** toda chamada registrada; nenhuma chamada "anônima".

## 9. Custo e quota

- Tabela de preços por modelo em configuração versionada (não hardcoded).
- Antes da chamada: reserva de quota (`usage_records` + Redis).
- Depois: confirma consumo e grava `ai_interactions.costEstimate`.
- Falha: libera reserva.
- Orçamento diário por tenant e mensal por plano; excedeu → `QUOTA_EXCEEDED`.
- Dashboard administrativo com custo por tarefa, tenant e modelo.

## 10. Persistência

### `ai_jobs`

`{ _id, type, owner, tenantKey, inputRefs[], status: QUEUED|RUNNING|SUCCEEDED|FAILED|CANCELLED, attempts, resultRef?, errorCode?, createdAt, startedAt?, finishedAt? }`

### `ai_interactions`

`{ _id, jobId, taskId, provider, model, promptId, promptVersion, inputHash, outputHash?, tokensIn, tokensOut, latencyMs, costEstimate, status, createdAt }`

### `prompt_templates`

`{ _id, taskId, version, body, outputSchemaVersion, status, createdBy, createdAt, activatedAt? }`

## 11. Contratos de API

| Método | Rota | Permissão | Descrição |
|:--|:--|:--|:--|
| GET | `/ai/jobs/{jobId}` | dono | status, progresso, erro |
| POST | `/ai/jobs/{jobId}/cancel` | dono | cancela se ainda não executou |
| GET | `/admin/ai/costs` | PLATFORM_ADMIN | custo agregado |
| GET | `/admin/ai/prompts` | PLATFORM_ADMIN | versões e status |

Endpoints de negócio (geração de seção, análise) ficam nas specs dos módulos e enfileiram `ai-generation`.

### Erros

| code | HTTP | Quando |
|:--|:--|:--|
| `QUOTA_EXCEEDED` | 429 | limite do plano |
| `AI_PROVIDER_UNAVAILABLE` | 503 | provedor fora após retries |
| `AI_INVALID_OUTPUT` | 502 | schema inválido após reparo |
| `INSUFFICIENT_CONTEXT` | 422 | contexto insuficiente para responder |
| `AI_TIMEOUT` | 504 | timeout |
| `AI_JOB_NOT_FOUND` | 404 | job inexistente/sem acesso |

## 12. Critérios de aceite

```gherkin
Cenário: saída inválida não é persistida
  Dado um provedor que retorna JSON fora do schema
  Quando a tarefa executa
  Então o job termina FAILED com AI_INVALID_OUTPUT
  E nenhum conteúdo é salvo como sucesso

Cenário: isolamento de contexto
  Dado dois tenants com documentos distintos
  Quando o tenant A executa uma tarefa
  Então nenhum chunk do tenant B é recuperado
  E o prompt não contém dado do tenant B

Cenário: custo é registrado
  Dado uma chamada concluída
  Quando o job termina
  Então ai_interactions contém tokens e custo estimado
  E a quota do tenant foi decrementada

Cenário: injeção de prompt é tratada como dado
  Dado um documento com instrução maliciosa
  Quando a tarefa executa
  Então a instrução não altera o sistema
  E o evento é registrado para auditoria
```

## 13. Testes

- **Unitários:** validação de schema, cálculo de custo, montagem de contexto, detecção de injection.
- **Integração:** provider fake (sucesso, inválido, timeout, 429), retry, fallback, quota.
- **Avaliação:** conjunto dourado por tarefa com métricas de precisão e aderência ao schema.
- **Segurança:** cross-tenant no RAG, PII em log, prompt injection.
- **Custo:** teste de orçamento e de liberação de reserva.

## 14. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-048 | provedor principal e fallback de IA | alto | ADR específico (a criar) antes da Fase 1 |
| OPEN-024 | embeddings e vector store | alto | decidir com RAG |
| OPEN-025 | preços e margem por plano | alto | depende de SPEC-012 |
| OPEN-026 | retenção de prompts/entradas | médio | LGPD |

## 15. Definition of Done específico

- [ ] Nenhuma chamada sem registro em `ai_interactions`.
- [ ] Teste de cross-tenant no RAG verde.
- [ ] Teste de injection no CI.
- [ ] Custo por tarefa visível no admin.
- [ ] Prompt versions com aprovação registrada.

## 16. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
