# ADR-021 — Polyrepo: documentação, web e API em repositórios separados

- **Status:** Aceito
- **Data:** 2026-11-05
- **Decisor:** Product Owner (decisão direta: "monorepo NÃO")
- **Supersede:** ADR-011 (monorepo com pnpm workspaces) e PRD §9.1
- **Relacionado:** ADR-002, ADR-012, ADR-022, ADR-023

## Contexto

O PRD §9.1 sugeria monorepo com `apps/` e `packages/`. O Product Owner definiu que o frontend será implantado na Vercel e o backend em VPS própria, em repositórios separados. Monorepo não será usado.

## Decisão

Adotar **polyrepo no GitHub** com três repositórios independentes (privados, salvo decisão em contrário):

| Repositório | Conteúdo | Deploy |
|:--|:--|:--|
| `projeta` | documentação (PRD, ADRs, specs, planos) | nenhum |
| `projeta-web` | Next.js + TypeScript | Vercel |
| `projeta-api` | NestJS (API + worker) + infraestrutura | VPS |

Regras:

1. Nenhum código de aplicação neste repositório de documentação.
2. `projeta-api` é dono dos schemas Zod e publica o pacote privado `@projeta/contracts` (GitHub Packages) e o `openapi.json` a cada release.
3. `projeta-web` consome `@projeta/contracts` (versão exata) e um client gerado a partir do `openapi.json`.
4. Mudança incompatível de contrato exige major, `/api/v2` e PR coordenado nos dois repositórios.
5. Desenvolvimento local usa clones irmãos; infraestrutura (Mongo, Redis, MinIO, Mailpit) sobe por Docker Compose no `projeta-api`.

### Configuração no GitHub

- Owner: **`andrelobo`** (conta de usuário no GitHub; não é organização).
- Visibilidade: **pública** para os três repositórios (decisão do owner em 2026-11-05).
- Repositórios públicos exigem: licença definida (`OPEN-047`), secret scanning com push protection, Dependabot, CodeQL e revisão do que entra em `docs/` (roadmap e decisões ficam visíveis).
- Consequência operacional: a sessão `gh` local já está autenticada como `andrelobo` (owner). O usuário `lobojoe` é da VPS, não do GitHub (ver ADR-022).
- `main` protegida: PR obrigatório, revisão, checks verdes, histórico linear, sem force push.
- `CODEOWNERS` por área (frontend, backend, infra, docs).
- GitHub Environments `staging` e `production` com segredos e aprovação manual em produção.
- `projeta-api` publica `@projeta/contracts` no **GitHub Packages** em tags `contracts-vX.Y.Z`.
- `projeta-web` tem deploy pela Vercel **via GitHub Actions** (CLI), com preview por PR e produção por tag aprovada (ADR-024).
- Issues e PRs usam os IDs `SPEC-xxx` / `ADR-xxx` no título para rastreabilidade.

## Consequências

- Sem `pnpm -r`: cada repositório tem seu próprio `package.json`, lockfile e pipeline.
- Contratos passam a ser artefato versionado, não código compartilhado por caminho relativo.
- Duas esteiras de CI/CD (Vercel e VPS) e dois conjuntos de segredos.
- Isolamento de permissões: acesso ao repositório de infraestrutura não dá acesso ao frontend, e vice-versa.
- Custo: sincronização explícita de versão entre web e API; benefício: deploys independentes e superfície menor.

## Alternativas consideradas

- Monorepo com pnpm: **rejeitado pelo Product Owner**.
- Monorepo com Turborepo/Nx: rejeitado junto com o anterior.
- Quatro repositórios (contratos separados): complexidade sem ganho; o pacote é publicado pelo `projeta-api`.
