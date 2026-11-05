# SPEC-000 — Fundação da plataforma

| Campo | Valor |
|:--|:--|
| ID | SPEC-000 |
| Versão | 0.1 |
| Status | Aprovada (2026-11-05) |
| Fase | 0 |
| Dono | System Architect |
| Depende de | — |
| Requisitos cobertos | RNF-001..022, RNF-003, RNF-010, RNF-011, RNF-017, RNF-018 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Entregar a base técnica dos dois repositórios de aplicação — `projeta-api` (VPS) e `projeta-web` (Vercel) — com configuração tipada, tratamento de erro, logging, validação, OpenAPI, saúde, segurança HTTP, paginação, idempotência, ambiente local, CI e harness de testes. Não há monorepo (ADR-021).

Esta spec **não** implementa regra de negócio.

## 2. Fora de escopo

- Qualquer módulo funcional (identidade, oportunidades, projetos).
- Provedores definitivos de IA, pagamento, e-mail, OCR e storage.
- Interface de produto (apenas o shell técnico do frontend).
- Deploy em produção (FASE 0 termina em staging).

## 3. Requisitos cobertos

| Requisito | Como é atendido |
|:--|:--|
| RNF-001 arquitetura modular | polyrepo + fronteiras de módulo + lint de dependência |
| RNF-002 TypeScript strict | `strict: true`, `noUncheckedIndexedAccess`, sem `any` implícito |
| RNF-003 API versionada | prefixo `/api/v1`, política de breaking change |
| RNF-004/005/006 testes | Vitest, Testcontainers, Supertest, Playwright |
| RNF-007 autorização no backend | guards globais; nenhuma rota privada sem guard |
| RNF-008 isolamento multi-tenant | `TenantContext` + repositório tipado (SPEC-002) |
| RNF-009 assíncrono | BullMQ + workers (SPEC-000 §8) |
| RNF-010 logs estruturados | pino JSON + correlação |
| RNF-011 observabilidade | OTel + métricas + health |
| RNF-012 backup | política documentada e testada em staging |
| RNF-013 recuperação | runbook de restore |
| RNF-014 acessibilidade | baseline WCAG 2.1 AA no shell |
| RNF-015 responsividade | breakpoints e componentes base |
| RNF-016 i18n | catálogo de strings, pt-BR inicial |
| RNF-017 OpenAPI | geração a partir dos schemas Zod |
| RNF-018 evolução de dados | runner de migrações versionadas |
| RNF-019 idempotência | middleware + chave por operação |
| RNF-020 arquivos controlados | política em SPEC-010 |
| RNF-021 custo de IA | orçamento e ledger em SPEC-006 |
| RNF-022 falhas externas | timeout, retry, circuit breaker |

## 4. Decisões de implementação

1. **Configuração** carregada por schema Zod em `projeta-api/src/config`; ausência/invalidade → processo não sobe (fail-fast). O `projeta-web` valida suas variáveis no build da Vercel.
2. **Erros** no formato Problem Details (RFC 9457) com campos extras `code`, `requestId`, `errors[]`.
3. **Validação** de body/query/params por schema Zod; erro 422 com lista de campos.
4. **Logs** JSON, com `requestId`, `tenantKey`, `userId`, `route`, `status`, `durationMs`; corpo de requisição **não** é logado por padrão; PII redigida.
5. **Health**: `/health/live` (processo), `/health/ready` (Mongo, Redis, storage, fila), `/api/v1/meta` (versão, commit, ambiente).
6. **Paginação**: cursor opaco para listagens públicas; página para administração; limite máximo 100.
7. **Idempotência**: header `Idempotency-Key` obrigatório em POST de geração/exportação/cobrança; registro com TTL e resposta reproduzida.
8. **Feature flags**: coleção `feature_flags` com cache; flag `domain.health.enabled = false` no MVP.
9. **Migrações**: runner versionado com coleção `schema_migrations`; execução como job, nunca no boot.
10. **Contratos**: schemas em `projeta-api/packages/contracts`, publicados como `@projeta/contracts` (GitHub Packages) e `openapi.json`; `projeta-web` fixa a versão exata.
10. **Shell do frontend**: layout, tema, componentes de erro/vazio/carregamento, foco visível, contraste AA.

## 5. Modelo de dados (infra)

| Coleção | Campos | Observação |
|:--|:--|:--|
| `schema_migrations` | `_id`, `name`, `appliedAt`, `checksum` | controle de migração |
| `feature_flags` | `key`, `enabled`, `value`, `updatedBy`, `updatedAt` | cache Redis 60 s |
| `outbox` | `_id`, `type`, `payload`, `status`, `availableAt`, `attempts`, `createdAt` | publicado por worker |
| `idempotency_keys` | `key`, `tenantKey`, `route`, `requestHash`, `responseRef`, `expiresAt` | TTL index |

## 6. Contratos de infraestrutura

| Método | Rota | Auth | Resposta |
|:--|:--|:--|:--|
| GET | `/health/live` | pública | `{ "status": "ok" }` |
| GET | `/health/ready` | pública (rate limit) | `200` com checagens ou `503` |
| GET | `/api/v1/meta` | pública | `{ version, commit, env, apiVersion }` |
| GET | `/metrics` | interna (token) | Prometheus |

### Envelope de erro

```json
{
  "type": "https://projeta.app/errors/validation",
  "title": "Dados inválidos",
  "status": 422,
  "detail": "Um ou mais campos são inválidos.",
  "instance": "/api/v1/projects",
  "code": "VALIDATION_ERROR",
  "requestId": "01J...",
  "errors": [{ "field": "title", "code": "too_small", "message": "mínimo 3 caracteres" }]
}
```

### Envelope de lista

```json
{ "items": [], "nextCursor": null }
```

## 7. Segurança

- Helmet com CSP compatível com Next.js; HSTS em produção.
- CORS restrito à origem do frontend, com `credentials`.
- Limite de corpo: 1 MB JSON; uploads seguem SPEC-010.
- Rate limit global por IP e por usuário; endpoints públicos com limite mais rígido.
- Nenhum segredo em log, erro ou resposta.
- Cookies: `httpOnly`, `Secure`, `SameSite=Lax` (ADR-014).

## 8. Assíncrono

- BullMQ com Redis; conexão dedicada.
- Toda fila declara `attempts`, `backoff`, `timeout`, `removeOnComplete`.
- Worker registra `jobId`, `queue`, `attempt`, `durationMs`, `status`.
- DLQ inspecionável e reprocessável por administrador (SPEC-013).

## 9. CI/CD (FASE 0)

### 9.1 `projeta-api` (GitHub Actions)

1. `install --frozen-lockfile`
2. `typecheck`
3. `lint`
4. `test:unit`
5. `test:integration` (Testcontainers)
6. `build`
7. `audit` de dependências (falha em crítico)
8. geração e validação do OpenAPI (sem diff inesperado)
9. publicação de `@projeta/contracts` em tag `contracts-vX.Y.Z`
10. build da imagem → GHCR → deploy na VPS (staging automático; produção com aprovação)

### 9.2 `projeta-web` (GitHub Actions → Vercel CLI)

1. `install --frozen-lockfile`
2. `typecheck`
3. `lint`
4. `test:unit`
5. `test:e2e:smoke` (Playwright, preview)
6. `vercel pull` → `vercel build` → `vercel deploy --prebuilt`
7. verificação de que a versão de `@projeta/contracts` corresponde ao OpenAPI esperado

- PR → preview; `main` → staging; tag `v*` → produção com aprovação.
- Auto-deploy da integração Git da Vercel **desabilitado** (ADR-024).

### 9.3 Regras

- Deploy só com aprovação humana em produção (GitHub Environment `production`).
- Staging automático a partir da branch principal.
- Todo deploy é disparado por GitHub Actions; não há deploy manual (ADR-024).
- Nenhum repositório depende do outro por caminho relativo.

## 10. Critérios de aceite

```gherkin
Cenário: configuração inválida impede o boot
  Dado que a variável MONGODB_URI não está definida
  Quando a API inicia
  Então o processo encerra com código diferente de zero
  E o log indica o campo ausente sem expor segredos

Cenário: erro segue Problem Details
  Dado um POST /api/v1/projects com título vazio
  Quando a requisição é processada
  Então a resposta é 422
  E o corpo contém type, title, status, code, requestId e errors

Cenário: health ready reflete dependências
  Dado que o Redis está indisponível
  Quando GET /health/ready é chamado
  Então a resposta é 503
  E o corpo indica qual dependência falhou

Cenário: idempotência evita duplicidade
  Dado um POST com Idempotency-Key "k1"
  Quando a mesma requisição é repetida com a mesma chave
  Então a segunda resposta é idêntica à primeira
  E nenhum recurso adicional é criado
```

## 11. Testes

- **Unitários:** schema de config, serialização de erro, cálculo de cursor, middleware de idempotência.
- **Integração:** health com dependências reais (Testcontainers), migrations, outbox publisher.
- **E2E:** shell do frontend carrega, login page renderiza, erro 404/500 tem página amigável.
- **Segurança:** headers, CORS, limite de body, rate limit, ausência de segredo em log.

## 12. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-002 | ferramenta de migração | baixo | decidir na FASE 0 |
| OPEN-008 | provedor de observabilidade | médio | decidir antes de staging |
| OPEN-009 | política de retenção de logs | médio | junto com LGPD |
| ASM-001 | WCAG 2.1 AA como meta de acessibilidade | médio | validar com Product |

## 13. Definition of Done específico

- [ ] `docker compose -f infra/docker/compose.local.yml up` sobe Mongo, Redis, MinIO e Mailpit.
- [ ] `projeta-web` e `projeta-api` sobem separadamente, cada um com seu lockfile.
- [ ] API e web sobem com um comando; README atualizado.
- [ ] OpenAPI publicado em `/api/v1/docs`.
- [ ] Health e métricas funcionando.
- [ ] CI verde em PR de exemplo.
- [ ] Nenhum segredo versionado.
- [ ] Runbook de restore escrito e testado em staging.

## 14. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
