# ADR-011 — Monorepo com pnpm workspaces

- **Status:** Rejeitado (2026-11-05)
- **Data original:** 2026-11-05
- **Decisor original:** System Architect
- **Substituído por:** ADR-021 (polyrepo)
- **Relacionado:** PRD §9.1, ADR-002, ADR-022

## Contexto

O PRD §9.1 sugeria monorepo com `apps/` (web, api, worker) e `packages/` (contracts, shared, ai, config). A proposta inicial era pnpm workspaces, sem Turborepo.

## Decisão original (revogada)

Adotar pnpm workspaces com pacotes compartilhados e um único pipeline.

## Motivo da rejeição

O Product Owner definiu que o frontend será implantado na **Vercel** e o backend em **VPS própria**, em repositórios separados, com deploys independentes. Monorepo foi explicitamente rejeitado.

## Consequência

Esta decisão **não** está em vigor. A topologia vigente é:

- `projeta` (documentação);
- `projeta-web` (Next.js na Vercel);
- `projeta-api` (NestJS + worker na VPS).

Contratos são publicados como pacote versionado (`@projeta/contracts`) a partir do `projeta-api`.

## Alternativas consideradas na época

- npm workspaces; Nx; Turborepo. Todas descartadas junto com o modelo de monorepo.
