# Emendas ao PRD 1.0

Registro formal de mudanças aprovadas pelo Product Owner que alteram a baseline. O PRD permanece preservado como documento original; em caso de conflito, **a emenda prevalece**.

---

## Emenda 01 — Topologia de repositórios e deploy

- **Data:** 2026-11-05
- **Autoridade:** Product Owner
- **Altera:** §9.1 (Monorepo) e trechos de §9.2/§9.4/§9.5 que pressupõem monorepo
- **ADR:** ADR-021 (polyrepo) e ADR-022 (deploy)

### Decisão

1. **Não haverá monorepo.** O código será dividido em repositórios separados no **GitHub**:
   - `projeta` — documentação (PRD, ADRs, specs, planos), sem código de aplicação;
   - `projeta-web` — Next.js + TypeScript, implantado na **Vercel**;
   - `projeta-api` — NestJS (API + worker) + infraestrutura, implantado em **VPS própria**.
2. O `projeta-api` publica o pacote **`@projeta/contracts`** (GitHub Packages) e o `openapi.json` a cada release. Os repositórios são públicos; a imagem do GHCR permanece privada.
3. O `projeta-web` consome `@projeta/contracts` com versão exata e um client gerado do OpenAPI.
4. Desenvolvimento local usa clones irmãos; a infraestrutura local sobe por Docker Compose no `projeta-api`.
5. A topologia de deploy, rede, TLS, CI/CD e recuperação está em ADR-022.

### O que não muda

- Nenhum requisito funcional (RF), regra de negócio (RB), persona, princípio ou métrica.
- Stack: Next.js, TypeScript, NestJS, MongoDB Atlas, Redis/BullMQ, storage S3.
- Domínios CULTURE (MVP) e HEALTH (futuro).
- Multi-tenancy, segurança e LGPD.

### Consequência documental

- SPEC-000 passa a tratar “fundação dos dois repositórios” em vez de monorepo.
- Contratos deixam de ser caminho relativo (`packages/contracts`) e passam a ser artefato versionado.
- Cada repositório tem pipeline próprio; o `projeta` não tem deploy.

---

## Emenda 02 — Autenticação entre Vercel e VPS

- **Data:** 2026-11-05
- **Autoridade:** System Architect (proposta) — **pendente de aprovação**
- **Altera:** detalhamento de §11 (segurança)
- **ADR:** ADR-023

### Proposta

- Produção e staging usam o mesmo domínio registrável (`app.<dominio>` e `api.<dominio>`), com cookie `Domain=.<dominio>`, `Secure`, `HttpOnly`, `SameSite=Lax`.
- Previews da Vercel rodam em modo mock, sem dados reais.
- Se preview autenticado for necessário, adotar BFF no Next (decisão futura).

Enquanto não aprovada, a Emenda 02 é **proposta**, não vigente.
