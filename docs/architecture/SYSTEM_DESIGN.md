# SYSTEM DESIGN — PROJETA

**Versão:** 1.0
**Status:** Aprovado em 2026-11-06; base técnica das specs aprovadas
**Base:** PRD 1.0 + Emenda 01 · ADR-001 a ADR-010 (aceitos) · ADR-011 (rejeitado) · ADR-012 a ADR-025 (aceitos)
**Última atualização:** 2026-11-05

> Este documento descreve **como** o sistema será construído para atender ao PRD. Ele não substitui as especificações por módulo (`docs/specifications/`), que contêm contratos e critérios de aceite.

---

## 1. Contexto e restrições

| Restrição | Consequência arquitetural |
|:--|:--|
| Time pequeno, produto em validação | Monólito modular, um único deploy de API, sem microserviços |
| Multi-tenancy obrigatória desde a fundação | Todo recurso privado tem `ownerType/ownerId`; guard central |
| IA é diferencial e risco de custo | Camada AI isolada, com orçamento, cache, fallback e livro-razão |
| Conteúdo externo é hostil | PDFs/páginas tratados como dados; validação de saída e isolamento de prompt |
| Documentos jurídicos e pessoais | Storage privado, URL assinada, auditoria, retenção |
| Mercado brasileiro, pt-BR | Datas/fuso, CNPJ/CPF, LGPD, acessibilidade |
| Domínio HEALTH futuro | Fronteira de domínio explícita, feature flag, modelo de dados separável |
| Custo previsível | Filas, limites por plano, jobs idempotentes, observabilidade de custo |

---

## 2. Topologia de execução

```
                         ┌──────────────────────────────┐
                         │  projeta-web (Vercel)         │
                         │  RSC + client components     │
                         │  cookies httpOnly            │
                         └──────────────┬───────────────┘
                                        │ HTTPS /api/v1
                         ┌──────────────▼───────────────┐
                         │  projeta-api — api (VPS)      │
                         │  REST + OpenAPI              │
                         │  guards: auth, tenant,       │
                         │  role, entitlement, quota    │
                         └───┬───────────┬───────────┬──┘
                             │           │           │
              ┌──────────────▼──┐   ┌────▼─────┐   ┌─▼──────────────┐
              │ MongoDB Atlas   │   │ Redis    │   │ Object Storage │
              │ dados + search  │   │ cache +  │   │ S3-compatible  │
              │                 │   │ filas    │   │ privado        │
              └─────────────────┘   └────┬─────┘   └────────────────┘
                                          │ BullMQ
                         ┌────────────────▼────────────────┐
                         │  projeta-api — worker        │
                         │  processors por fila            │
                         └───────┬──────────────┬──────────┘
                                 │              │
                    ┌────────────▼───┐   ┌──────▼──────────┐
                    │ LLM providers  │   │ OCR / e-mail /  │
                    │ (abstratos)    │   │ pagamento       │
                    └────────────────┘   └─────────────────┘
```

**Decisões de topologia:**

- Um único banco MongoDB (Atlas) com coleções por agregado; sem sharding no MVP.
- Redis usado para cache, rate limit distribuído, filas e contadores de quota.
- Workers escalam horizontalmente por fila; API não executa trabalho pesado.
- Nenhum worker acessa o banco sem `TenantContext` quando o job é de tenant.

---

## 3. Repositórios (polyrepo no GitHub)

**Não há monorepo** (ADR-021, Emenda 01). Três repositórios independentes e públicos no GitHub, sob o owner `andrelobo`:

| Repositório | Conteúdo | Deploy |
|:--|:--|:--|
| `projeta` | documentação: PRD, ADRs, specs, planos | nenhum |
| `projeta-web` | Next.js + TypeScript | Vercel |
| `projeta-api` | NestJS (API + worker) + infraestrutura | VPS |

### 3.1 `projeta-api`

```
projeta-api/
├── src/
│   ├── main.ts                  # bootstrap HTTP
│   ├── worker.ts                # bootstrap dos processors
│   ├── modules/                 # módulos NestJS (fronteiras do §4)
│   ├── shared/                  # Money, datas, erros, Result, logger
│   ├── ai/                      # providers, prompts, RAG, custo
│   └── config/                  # env schema + loader tipado
├── packages/
│   └── contracts/               # Zod schemas + tipos + openapi.json (publicado)
├── test/                        # unit, integração, fixtures
├── infra/
│   ├── docker/                  # Dockerfile, compose local/prod
│   ├── caddy/                   # reverse proxy e TLS
│   └── scripts/                 # migração, seed, backup, deploy
├── .github/workflows/           # CI + release + deploy
├── package.json
└── pnpm-lock.yaml
```

### 3.2 `projeta-web`

```
projeta-web/
├── src/
│   ├── app/                     # App Router (RSC + client components)
│   ├── components/
│   ├── lib/
│   │   ├── api/                 # client gerado do OpenAPI
│   │   └── auth/                # leitura de sessão e guards visuais
│   └── styles/
├── public/
├── .github/workflows/           # lint, typecheck, testes
├── package.json
└── pnpm-lock.yaml
```

### 3.3 `projeta` (este repositório)

Apenas `docs/` e arquivos de processo. **Nenhum código de aplicação.** Não tem pipeline de deploy.

### 3.4 Contratos entre repositórios

- Fonte de verdade: schemas Zod em `projeta-api/packages/contracts`.
- Publicação: pacote `@projeta/contracts` no GitHub Packages, em tags `contracts-vX.Y.Z` (repositórios públicos; imagem GHCR privada).
- `openapi.json` é gerado no release da API e anexado ao pacote/artefato.
- `projeta-web` fixa versão exata no `package.json`, gera o client a partir do OpenAPI e usa `NODE_AUTH_TOKEN` (`read:packages`) no install. Ver `docs/implementation/GITHUB_PACKAGES.md`.
- Mudança incompatível: major + `/api/v2` + PR coordenado nos dois repositórios.
- CI do web falha se a versão do contrato divergir do OpenAPI esperado.

### 3.5 Regras de dependência

- Nenhum import por caminho relativo entre repositórios.
- `contracts` não depende de framework.
- `shared` não depende de `contracts`.
- `ai` depende de `contracts` e `shared`.
- Módulos de `src/modules` só se comunicam pelo serviço público do módulo, nunca pelo repositório alheio.

### 3.6 Desenvolvimento local

```
~/dev/projeta/           # clones irmãos
├── projeta/             # docs
├── projeta-api/         # API + worker + infra
└── projeta-web/         # Next.js
```

- `projeta-api`: `docker compose -f infra/docker/compose.local.yml up -d` (Mongo, Redis, MinIO, Mailpit) e `pnpm dev`.
- `projeta-web`: `.env.local` com `NEXT_PUBLIC_API_URL=http://localhost:3001/api/v1`.
- CORS local permite `http://localhost:3000`.
- Nenhum segredo compartilhado entre os repositórios.

---

## 4. Fronteiras de módulos (API)

```
IdentityModule ──► UsersModule
       │
       ├──► OrganizationsModule ──► MembershipsModule
       │
       ├──► ProfilesModule
       ├──► OpportunitiesModule ──► OpportunitySourcesModule
       ├──► DocumentsModule ──► EditalAnalyzerModule ──► AIModule
       ├──► EligibilityModule
       ├──► ProjectsModule ──► ProjectVersionsModule
       │        └──► BudgetsModule ──► ActivitiesModule
       ├──► SubscriptionsModule ──► EntitlementsModule (derivado: quotas/features)
       ├──► NotificationsModule
       ├──► AuditModule
       └──► AdminModule
```

**Regras:**

1. Cada módulo expõe um `*.service.ts` público e mantém repositórios internos.
2. Comunicação síncrona entre módulos apenas via serviço público; nunca via repositório alheio.
3. Efeitos colaterais entre módulos via **domain events** internos (in-process) e, quando precisar de garantia, **outbox**.
4. `AIModule` não conhece regra de negócio de projeto: recebe contexto montado pelo módulo chamador.
5. `AuditModule` é consumidor de eventos e não é chamado dentro de transação de negócio.

---

## 5. Ciclo de vida de uma requisição

```
HTTP
 └─ CorrelationIdMiddleware        (x-request-id, gera se ausente)
 └─ RequestLogMiddleware           (método, rota, status, duração, tenant)
 └─ Helmet / CORS / body limit
 └─ JwtAuthGuard                   (access token do cookie ou Bearer)
 └─ ThrottlerGuard (Redis)         (por IP + por usuário autenticado)
 └─ TenantGuard                    (resolve ownerType/ownerId do recurso)
 └─ RolesGuard                     (papel mínimo no tenant)
 └─ EntitlementGuard               (feature do plano)
 └─ UsageQuotaGuard                (limite do período; reserva atômica)
 └─ ValidationPipe (Zod)           (body, query, params)
 └─ Controller → ApplicationService → Domain → Repository
 └─ Response envelope / ProblemDetails
```

**Regras de ouro:**

- Autorização **sempre** no servidor; o frontend só esconde o que já é proibido.
- `TenantGuard` **não confia** em `organizationId` do cliente: resolve o recurso e valida o membership.
- Toda escrita relevante gera `audit_log` (quem, o quê, antes/depois resumido, IP, user-agent).
- Toda operação cara (IA, export, OCR) é assíncrona com `jobId` e status consultável.

---

## 6. Camadas dentro de cada módulo

```
projeta-api/src/modules/<module>/
├── <module>.module.ts
├── <module>.controller.ts        # HTTP, DTO, sem regra
├── <module>.service.ts           # casos de uso (público)
├── application/                  # handlers, orquestração
├── domain/                       # entidades, value objects, regras puras
├── infrastructure/               # schemas Mongoose, repositórios, adapters
└── dto/                          # schemas Zod + tipos derivados
```

**Testabilidade:** regra de negócio em `domain/` sem dependência de Nest, Mongo ou HTTP. Casos de uso testáveis com repositórios em memória.

---

## 7. Persistência

### 7.1 Convenções

- `_id`: ObjectId.
- `createdAt`, `updatedAt` em UTC.
- `deletedAt` para soft delete (quando aplicável).
- `schemaVersion` em documentos versionados.
- Campos monetários: `Decimal128` (BSON), convertidos para `Money` na borda.
- Toda coleção privada tem `ownerType` e `ownerId`.

### 7.2 Índices mínimos (além dos definidos no PRD §8.4)

| Coleção | Índice | Motivo |
|:--|:--|:--|
| `opportunities` | `{ domain: 1, status: 1, closingDate: 1 }` | radar e prazos |
| `opportunities` | `{ sourceId: 1, externalId: 1 }` unique parcial | deduplicação por fonte |
| `opportunity_evidence` | `{ opportunityId: 1, field: 1 }` | auditoria por campo |
| `projects` | `{ "owner.ownerId": 1, status: 1, updatedAt: -1 }` | listagem |
| `project_versions` | `{ proposalId: 1, versionNumber: -1 }` unique | histórico |
| `documents` | `{ ownerType: 1, ownerId: 1, category: 1 }` | biblioteca |
| `ai_interactions` | `{ tenantKey: 1, createdAt: -1 }` | custo por tenant |
| `usage_records` | `{ tenantKey: 1, metric: 1, period: 1 }` unique | quota |
| `notifications` | `{ userId: 1, readAt: 1, createdAt: -1 }` | central |
| `audit_logs` | `{ tenantKey: 1, createdAt: -1 }` | consulta |
| `outbox` | `{ status: 1, availableAt: 1 }` | publicação |

Índices só entram em produção com plano de criação e métrica de uso.

### 7.3 Regra de tenant no repositório

Todo repositório de recurso privado recebe `TenantContext` obrigatório e aplica filtro no `find`/`update`. O tipo do repositório **impede** chamada sem contexto (parâmetro não opcional). Testes de IDOR cobrem cada repositório.

---

## 8. Processamento assíncrono

### 8.1 Filas (BullMQ)

| Fila | Produtor | Consumidor | Idempotência |
|:--|:--|:--|:--|
| `opportunity-discovery` | admin/cron | worker | `sourceId + externalId + runId` |
| `opportunity-verification` | cron/admin | worker | `opportunityId + attemptedAt` (janela) |
| `document-processing` | upload | worker | `documentId + hash` |
| `pdf-extraction` | document-processing | worker | `documentId + hash` |
| `ocr-processing` | pdf-extraction | worker | `documentId + pageRange` |
| `edital-analysis` | upload/reprocess | worker | `analysisId + inputHash` |
| `ai-generation` | API | worker | `jobId` (client idempotency key) |
| `document-export` | API | worker | `versionId + format` |
| `notifications` | eventos | worker | `eventId + channel` |
| `billing-webhooks` | provedor | worker | `providerEventId` unique |

### 8.2 Regras

- Retry com backoff exponencial e limite por fila; após o limite → DLQ com payload e erro.
- Timeout por job; jobs longos reportam progresso.
- Nenhum job altera estado sem verificar idempotência.
- Outbox para eventos de negócio que precisam sobreviver a falha entre commit e publicação.
- DLQ tem painel administrativo e rotina de reprocessamento manual (RF-125).

---

## 9. Camada AI

```
AIModule
├── provider registry (OpenAI, Anthropic, DeepSeek, ...)  ← provedor em aberto
├── prompt registry (id + versão + hash)
├── context assembler (orçamento de tokens, RAG, tenant)
├── schema validation (Zod → retry → fallback)
├── guardrails (prompt injection, PII, output limits)
├── cost ledger (tokens, modelo, duração, custo)
└── job orchestration (fila ai-generation)
```

**Contrato interno:** `runTask(taskId, input, ctx) → Result<ValidatedOutput>`.

- Toda operação declara `taskId` (AI-001..AI-010) e versão de prompt.
- Toda saída é validada; falha de schema não é persistida como sucesso.
- Toda chamada registra `ai_interactions` com hash das entradas (RB-010).
- Orçamento por tenant/dia e por operação; excedeu → erro tratado, não truncamento silencioso.
- Conteúdo externo entra sempre delimitado (`<documento>…</documento>`) e com instrução explícita de que é dado.

Detalhe em `docs/specifications/SPEC-006-ai-platform.md`.

---

## 10. Arquivos e storage

- Bucket privado por ambiente; prefixo `tenant/{ownerType}/{ownerId}/`.
- Upload direto ao storage via URL pré-assinada **ou** via API (arquivos pequenos), sempre com validação de MIME real, tamanho e hash.
- Download sempre por URL assinada de curta duração; nunca URL pública.
- Antivírus/varredura antes de disponibilizar (hook na fila `document-processing`).
- Metadados em `documents`; binário no storage; hash SHA-256 para integridade e deduplicação.
- Exclusão: soft delete no metadado + política de retenção para o binário.

---

## 11. Busca

- MongoDB Atlas Search no MVP, índice por coleção (`opportunities`, `documents`, `projects`).
- Campos pesquisáveis de oportunidade: título, descrição, organizador, áreas, texto extraído do edital.
- Filtros estruturados usam índices compostos; busca textual usa Atlas Search.
- Reindexação é job administrativo, não operação de request.
- Elasticsearch/OpenSearch só com ADR e evidência de gargalo.

---

## 12. Segurança

### 12.1 Modelo de ameaça (resumo)

| Ameaça | Controle |
|:--|:--|
| Acesso a recurso alheio (IDOR) | tenant guard + repositório com contexto obrigatório + testes |
| Vazamento cross-tenant na IA | filtro de tenant no RAG e no context assembler |
| Prompt injection em edital | delimitação de dados, sem execução de ferramenta, validação de saída |
| Upload malicioso | MIME real, limite, hash, varredura, storage privado |
| Força bruta / abuso | rate limit por IP e usuário, lockout progressivo, CAPTCHA em endpoints públicos |
| CSRF | cookie SameSite + token CSRF em mutações |
| XSS | escaping no frontend, CSP, sanitização de HTML renderizado |
| NoSQL injection | validação de schema, proibição de operadores em input cru |
| Webhook falso | assinatura HMAC + idempotência + janela de tempo |
| Exposição de segredo | cofre de secrets, nunca em log, rotação |
| Exfiltração por export | autorização por versão + auditoria + limite de download |
| SSRF em fontes/conectores | allowlist de domínio, bloqueio de IP privado, timeout e sem redirect para rede interna |
| Abuso de scraping/API | rate limit por token/tenant, quotas e detecção de padrão anômalo |

### 12.2 Autenticação

- Senha com **Argon2id** (parâmetros definidos na FASE 0).
- Access token JWT curto (ex.: 15 min) + refresh token rotativo em cookie `httpOnly`, `Secure`, `SameSite=Lax`.
- Refresh token armazenado por hash, com família e revogação; reutilização de refresh revoga a família.
- Verificação de e-mail obrigatória antes de funcionalidades pagas.
- Recuperação de senha com token de uso único, TTL curto, resposta idêntica para e-mail existente ou não.

### 12.3 Autorização

- RBAC por tenant (`OWNER > ADMIN > EDITOR > VIEWER`) + regra de propriedade.
- Capacidades declaradas em matriz única (`docs/specifications/SPEC-002`), usada por backend e frontend.
- Nenhuma rota privada sem guard; teste automatizado verifica que toda rota tem guard.
- Operações administrativas exigem papel de plataforma (`PLATFORM_ADMIN`), separado de papéis de organização.

### 12.4 LGPD

- Base legal e finalidade por categoria de dado documentadas.
- Direitos do titular: acesso, correção, portabilidade, eliminação (com retenção legal).
- Dados sensíveis (HEALTH) fora do MVP; quando entrarem, exigem DPIA e isolamento.
- Subprocessadores (IA, storage, e-mail, pagamento) registrados com região e retenção.

---

## 13. Observabilidade

| Sinal | Implementação |
|:--|:--|
| Logs | JSON estruturado (pino), com `requestId`, `tenantKey`, `userId`, `route`, `latency` |
| Métricas | OpenTelemetry + Prometheus: RED (rate, errors, duration) por rota; USE por fila |
| Traces | OTel ponta a ponta, incluindo job e chamada de IA |
| Filas | profundidade, idade, falhas, DLQ |
| IA | tokens, custo, latência, taxa de schema inválido, fallback |
| Negócio | oportunidades verificadas, análises concluídas, projetos que avançaram (North Star) |
| Alertas | erro 5xx, DLQ > 0, custo de IA acima do orçamento, falha de webhook, backup atrasado |

---

## 14. Desempenho e capacidade

Metas (PRD §12):

- p95 < 500 ms para consultas simples de API.
- p95 < 1 s para busca comum.
- Jobs pesados assíncronos, com status.
- Disponibilidade pretendida: 99,5% mensal.

Implicações:

- Paginação obrigatória em listagens (cursor para oportunidades; página para admin).
- Projeções explícitas no Mongo (sem `SELECT *`).
- Cache Redis para catálogo de planos, oportunidades em destaque e contadores.
- Exportação e análise nunca no ciclo de request.

---

## 15. Falhas e degradação

| Falha | Comportamento |
|:--|:--|
| Provedor de IA indisponível | job falha com retry; UI mostra estado "tentando novamente"; fallback opcional por ADR |
| OCR indisponível | análise marcada como `PARTIAL`, PDF escaneado sinalizado |
| Storage indisponível | upload falha com erro claro; metadado não é criado |
| Redis indisponível | API degrada: sem cache e sem fila → enfileiramento falha com erro explícito (não silencioso) |
| Mongo indisponível | 503 com `Retry-After`; workers pausam consumo |
| Webhook duplicado | idempotência por `providerEventId` |
| Fonte fora do ar | verificação marcada como `FAILED`; oportunidade não é promovida a `OPEN` |

---

## 16. Ambientes e deploy

| Ambiente | Uso | Dados |
|:--|:--|:--|
| `local` | desenvolvimento | docker compose (Mongo, Redis, MinIO, Mailpit) · web `localhost:3000` · api `localhost:3001` |
| `staging` | homologação e E2E | Vercel (staging) + VPS (compose de staging); dados sintéticos |
| `production` | clientes | Vercel (produção) + VPS (compose de produção); dados reais |

| Componente | Onde roda |
|:--|:--|
| `projeta-web` | Vercel (CDN + funções) |
| `projeta-api` (HTTP) | VPS, container atrás de Caddy |
| `projeta-api` (worker) | VPS (usuário `lobojoe`), container sem porta exposta |
| Redis | VPS, container interno |
| MongoDB | Atlas (externo) |
| Object storage | S3-compatible (externo) |
| Contratos | GitHub Packages (`@projeta/contracts`) |

- API e worker na mesma imagem, comandos diferentes.
- Migrações de dados versionadas (`docs/implementation`), executadas como job, nunca no boot.
- Feature flags para liberar módulo por plano/tenant.
- Segredos em GitHub Environments e no `.env` do VPS (permissão 600); nunca versionados.
- Deploy da API por tag + healthcheck; rollback para a tag anterior.
- Web e API são implantados por **GitHub Actions** (ADR-024): Vercel via CLI, API via GHCR + SSH. Auto-deploy da integração Git da Vercel desabilitado.
- Migrações executadas como job único antes do rollout; nunca no boot do container.

---

## 17. Decisões em aberto que afetam esta arquitetura

Ver `docs/validation/OPEN_DECISIONS.md`. As principais:

- provedor de IA principal e política de fallback;
- provedor de pagamento e modelo de assinatura;
- storage definitivo e região;
- OCR (serviço vs biblioteca);
- e-mail transacional;
- Atlas Search vs busca nativa no MVP;
- limites finais dos planos;
- estratégia de autenticação em previews da Vercel (`OPEN-051`).

Nenhuma dessas decisões pode ser resolvida por suposição em código: exigem ADR.
