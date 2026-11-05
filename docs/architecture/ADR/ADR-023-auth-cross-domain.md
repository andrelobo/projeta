# ADR-023 — Autenticação entre Vercel e VPS (domínios e previews)

- **Status:** Proposto — requer aprovação
- **Data:** 2026-11-05
- **Decisor:** System Architect
- **Relacionado:** ADR-014, ADR-021, ADR-022

## Contexto

Com frontend na Vercel e API na VPS, o cookie de sessão precisa atravessar origens. Se `app` e `api` estiverem em domínios diferentes (ex.: `*.vercel.app` e IP da VPS), o cookie `SameSite=Lax` não é enviado em chamadas do navegador; `SameSite=None` depende de cookies de terceiros, cada vez mais bloqueados.

## Decisão

1. **Produção e staging usam o mesmo domínio registrável:** `app.<dominio>` (Vercel) e `api.<dominio>` (VPS). Cookie `Domain=.<dominio>`, `Secure`, `HttpOnly`, `SameSite=Lax`. CORS com allowlist e `credentials: true`.
2. **SSR do Next** chama a API server-to-server, repassando o cookie; o navegador não faz chamada cross-site para a API em fluxo autenticado.
3. **Preview deployments** (`*.vercel.app`) rodam em **modo mock** (`NEXT_PUBLIC_USE_MOCKS=1`), sem login real e sem dados de produção. Previews servem para revisão de UI.
4. Se preview autenticado for necessário no futuro, adotar **BFF** no Next (route handlers) com tokens em cookie do domínio da Vercel e `Authorization` para a API — decisão futura, não MVP.
5. CSRF continua obrigatório (double-submit) mesmo com `SameSite=Lax`.

## Consequências

- Exige domínio próprio e DNS apontando `app` para a Vercel e `api` para a VPS.
- Ambiente local usa `localhost:3000` e `localhost:3001` (mesmo site; portas ignoradas).
- Previews não testam autenticação real; testes E2E autenticados rodam em staging.
- `COOKIE_DOMAIN` vazio em local, `.<dominio>` em produção.
- Sem `SameSite=None`, sem depender de cookie de terceiro.

## Alternativas consideradas

- `SameSite=None` para previews: rejeitado por bloqueio de terceiros.
- BFF desde o início: mais robusto, porém adiciona função Vercel em todo request; fica como fallback.
- Token em `localStorage`: rejeitado por XSS.
