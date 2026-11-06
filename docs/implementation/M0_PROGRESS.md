# Progresso do M0 — Fundação

**Data:** 2026-11-06
**Papéis:** Backend Engineer (`projeta-api`) e Frontend Engineer (`projeta-web`)
**Escopo:** M0.1 (bootstrap), M0.2 (config tipada) e M0.3 (API shell), conforme SPEC-000 aprovada.
**Fora de escopo:** funcionalidade de produto, banco, fila, storage, CI e deploy.

> Todo o trabalho foi entregue por Pull Request e mesclado na `main`:
>
> - `projeta#1`, `projeta-api#1`, `projeta-web#1` — M0.1/M0.2 e reconciliação documental;
> - `projeta-api#2` — M0.3 (health, Problem Details e request id).

---

## 1. M0.1 — Bootstrap dos repositórios

| Item | `projeta-api` | `projeta-web` |
|:--|:--|:--|
| Gerenciador | pnpm 11.15.1 (`packageManager`) | pnpm 11.15.1 (`packageManager`) |
| Node | 22 (`.nvmrc`, `engines >=22`) | 22 (`.nvmrc`, `engines >=22`) |
| TypeScript | 5.9.3, `strict` + `noUncheckedIndexedAccess` | 5.9.3, `strict` + `noUncheckedIndexedAccess` |
| Lint | ESLint 9 flat config + typescript-eslint | ESLint 9 flat config + `eslint-config-next` |
| Formatação | Prettier + `.editorconfig` | Prettier + `.editorconfig` |
| Testes | Vitest 3.2.7 (+ Supertest no M0.3) | Vitest 3.2.7 |
| Lockfile | `pnpm-lock.yaml` | `pnpm-lock.yaml` |
| Licença | Apache-2.0 (`LICENSE`) | Apache-2.0 (`LICENSE`) |

Ambos os repositórios têm `.gitignore` cobrindo dependências, build, cobertura e arquivos de ambiente. Nenhum `.env` real foi criado ou versionado.

## 2. M0.2 — Config tipada

### `projeta-api`

- `src/config/env.ts`: schema Zod com `NODE_ENV`, `PORT`, `MONGODB_URI`, `REDIS_URL`, `CORS_ORIGIN`, `LOG_LEVEL` e `OTEL_EXPORTER_OTLP_ENDPOINT`.
- Fail-fast: `loadEnv()` lança `ConfigValidationError` com o **campo** e o motivo, sem ecoar valores.
- O bootstrap valida o ambiente antes de criar a aplicação Nest.
- `src/config/config.module.ts`: provider global `APP_CONFIG` para injeção tipada.
- `.env.example` documenta as variáveis sem segredo real.

### `projeta-web`

- `src/config/env.ts`: schema Zod com `NEXT_PUBLIC_APP_ENV` e `NEXT_PUBLIC_API_URL`.
- `next.config.ts` chama `loadWebEnv(process.env)`: build inválido falha antes de gerar artefato.
- Shell mínimo do App Router com `lang="pt-BR"`, foco visível, contraste e respeito a `prefers-reduced-motion`.
- `.env.example` documenta as variáveis públicas.

## 3. M0.3 — API shell

| Entrega | Implementação |
|:--|:--|
| `GET /health/live` | responde `200 { "status": "ok" }` sem consultar dependências |
| `GET /health/ready` | agrega indicadores e responde `200` ou `503` com `checks` por dependência |
| Indicadores | verificação TCP de Mongo e Redis, com timeout de 1 s |
| Problem Details | filtro global RFC 9457 com `type`, `title`, `status`, `detail`, `instance`, `code`, `requestId` e `errors[]` |
| Correlação | middleware `x-request-id` (UUID gerado ou valor válido do cliente) |
| Segurança HTTP | CORS restrito a `CORS_ORIGIN`, body JSON de 1 MB, `x-powered-by` removido |
| Testes | Vitest + Supertest, incluindo E2E do shell |

**Limite consciente:** a readiness usa TCP, não ping autenticado. Os clientes reais de Mongo e Redis substituem os indicadores em M0.5/M0.8; storage e fila entram quando existirem.

## 4. Evidências executadas

### `projeta-api`

```text
pnpm typecheck   → 0 erros
pnpm lint        → 0 erros
pnpm test        → 7 arquivos, 32 testes aprovados
pnpm build       → dist/main.js gerado
smoke /health/live   → 200 {"status":"ok"}
smoke /health/ready  → 503 com mongodb e redis "down"
smoke rota inexistente → 404 application/problem+json com requestId
boot sem MONGODB_URI  → exit 1, log cita o campo (sem valor de segredo)
```

### `projeta-web`

```text
pnpm typecheck   → 0 erros (após next typegen)
pnpm lint        → 0 erros
pnpm test        → 1 arquivo, 5 testes aprovados
pnpm build sem NEXT_PUBLIC_API_URL → falha explícita em next.config.ts
pnpm build com NEXT_PUBLIC_API_URL  → build estático concluído (rota /)
```

## 5. Decisões de implementação registradas

1. **pnpm 11 e GitHub Packages.** O pnpm ignora `_authToken=${NODE_AUTH_TOKEN}` em `.npmrc` de projeto. Os arquivos versionados mantêm somente `@projeta:registry`; o token deve ser gravado em `~/.npmrc` no install da Vercel/CI. Ver `GITHUB_PACKAGES.md`.
2. **Porta do web.** `pnpm dev` usa **3001**, para não colidir com a API em 3000.
3. **Typecheck do web.** O comando roda `next typegen && tsc --noEmit`; as definições de rota do Next são geradas antes da checagem.
4. **Tipo de entrada da config.** `loadEnv` aceita `Record<string, string | undefined>` em vez de `NodeJS.ProcessEnv`, porque `@types/node` 26 exige `NODE_ENV` na interface.
5. **Injeção explícita.** `HealthController` usa `@Inject(HealthService)` para não depender de `design:paramtypes`; o Vitest (esbuild) não emite metadados de decorador.
6. **`detail` de erro.** O filtro usa apenas o campo `detail` do payload; mensagens padrão do Nest/Express (“Cannot GET…”) não vazam para a resposta.
7. **Nenhuma dependência de `@projeta/contracts` ainda.** O escopo está configurado, mas o pacote só passa a ser consumido no M0.11.

## 6. Pendências manuais (exigem a conta `andrelobo`)

| Item | Ação |
|:--|:--|
| PAT de leitura | criar token com `read:packages` |
| Vercel | cadastrar `NODE_AUTH_TOKEN` nos três ambientes e ajustar o Install Command |
| GitHub Actions | `gh secret set NODE_AUTH_TOKEN --repo andrelobo/projeta-web` |
| Branch protection | aplicar regras em `main` dos dois repositórios (sugestão abaixo) |

Sugestão de proteção inicial para `main` (sem exigir status checks, que ainda não existem):

```bash
for repo in projeta-api projeta-web; do
  gh api -X PUT "repos/andrelobo/$repo/branches/main/protection" \
    --input - <<'JSON'
{
  "required_status_checks": null,
  "enforce_admins": false,
  "required_pull_request_reviews": null,
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "required_linear_history": true
}
JSON
done
```

## 7. Próximo passo

**M0.4 — Web shell:** layout e tema, páginas de erro (`404`/`500`), estados de carregamento/vazio e base de acessibilidade, com testes.
