# ADR-024 — Deploy via GitHub Actions para Vercel e VPS

- **Status:** Aceito
- **Data:** 2026-11-05
- **Decisor:** Product Owner
- **Supersede:** mecanismo de entrega do ADR-022 (a topologia Vercel + VPS permanece)
- **Relacionado:** ADR-021, ADR-022, ADR-023

## Contexto

O owner definiu que **todo deploy é disparado pelo GitHub**, tanto para a Vercel quanto para a VPS. A integração automática da Vercel com o repositório criaria um segundo caminho de deploy, sem aprovação e sem rastreabilidade única.

## Decisão

1. **Nenhum deploy manual.** Todo deploy nasce de um workflow no GitHub Actions, no repositório correspondente.
2. **Vercel** é acionada por CLI (`vercel pull` → `vercel build` → `vercel deploy --prebuilt`), com token em GitHub Secrets. O auto-deploy da integração Git da Vercel é **desabilitado**.
3. **VPS** recebe a imagem publicada no GHCR e é atualizada por SSH, com migração, healthcheck e rollback.
4. **GitHub Environments** `staging` e `production`; produção exige aprovação manual.
5. **Fluxo de branches:**
   - PR → CI + preview da Vercel (sem produção);
   - push em `main` → deploy automático de **staging** (web + API);
   - tag `v*` → **produção** com aprovação.
6. **Segurança:** permissões mínimas por workflow, actions fixadas por SHA, host key verificada, deploy user sem root, segredos apenas em Environments.
7. **Repositórios públicos:** previews de PR de fork não recebem segredos; workflows usam `pull_request` (nunca `pull_request_target`); imagem do GHCR permanece **privada** e a VPS autentica com token de `read:packages`; secret scanning e push protection habilitados.

## Workflows

### `projeta-web`

| Workflow | Gatilho | Ações |
|:--|:--|:--|
| `ci.yml` | PR e push | install, typecheck, lint, test, build |
| `deploy.yml` | push `main` / tag `v*` | install, build, `vercel pull/build/deploy --prebuilt` |

- Staging usa o projeto Vercel `projeta-web-staging`; produção usa `projeta-web`.
- `main` → staging; tag → produção (Environment `production` com revisores).

### `projeta-api`

| Workflow | Gatilho | Ações |
|:--|:--|:--|
| `ci.yml` | PR e push | install, typecheck, lint, test:unit, test:integration, build, OpenAPI |
| `release.yml` | tag `v*` | build/push GHCR, migração, deploy, healthcheck, rollback |
| `contracts.yml` | tag `contracts-v*` | publicar `@projeta/contracts` no GitHub Packages |

Sequência do `release.yml`:

1. build da imagem com tag `vX.Y.Z` e digest;
2. push para `ghcr.io/andrelobo/projeta-api`;
3. SSH na VPS → `docker compose pull`;
4. `docker compose run --rm api migrate` (em `/home/lobojoe/projeta`);
5. `docker compose up -d` (api + worker);
6. healthcheck com retry;
7. falha → rollback para a tag anterior registrada em `.deploy/previous-tag`.

### Segredos (nomes)

| Secret | Onde | Uso |
|:--|:--|:--|
| `VERCEL_TOKEN` | Environment | CLI |
| `VERCEL_ORG_ID` | Environment | CLI |
| `VERCEL_PROJECT_ID_STAGING` / `VERCEL_PROJECT_ID_PROD` | Environment | projeto por ambiente |
| `VPS_HOST` / `VPS_USER` | Environment | SSH; `VPS_USER=lobojoe` |
| `VPS_SSH_KEY` / `VPS_KNOWN_HOSTS` | Environment | SSH seguro |
| `GITHUB_TOKEN` | workflow | GHCR (`packages: write`) |
| `GHCR_READ_TOKEN` | Environment + VPS | pull da imagem privada (`read:packages`) |
| `NODE_AUTH_TOKEN` | Vercel + GitHub Actions | leitura de `@projeta/contracts` no GitHub Packages (`read:packages`) |

## Consequências

- Um único caminho auditável de deploy; o histórico fica no GitHub.
- A Vercel deixa de deploiar sozinha: desligar a integração Git é obrigatório.
- Exige manter tokens e IDs de projeto nos Environments.
- Rollback é explícito: Vercel (`vercel rollback`/redeploy) e API (tag anterior).
- Custo: mais um workflow para manter; benefício: aprovação, trilha e padronização.

## Alternativas consideradas

- Integração Git da Vercel (auto-deploy): rejeitada por criar caminho paralelo sem aprovação.
- Deploy manual por CLI/SSH: rejeitado por falta de trilha e reprodutibilidade.
- Runner self-hosted na VPS: possível no futuro; não necessário no MVP.
