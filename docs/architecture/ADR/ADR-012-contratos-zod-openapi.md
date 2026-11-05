# ADR-012 — Contratos primeiro: Zod como fonte de verdade + OpenAPI gerado

- **Status:** Aceito (2026-11-05, aprovado com o plano)
- **Data:** 2026-11-05
- **Decisores:** System Architect
- **Relacionado:** ADR-001, RNF-017

## Contexto

Frontend, backend, workers e IA compartilham formatos. Duplicar tipos gera divergência silenciosa. O PRD exige DTOs, validação e OpenAPI.

## Decisão

Definir os schemas no pacote `@projeta/contracts` (código-fonte em `projeta-api/packages/contracts`) com **Zod**; derivar tipos TypeScript, validar na API (pipe) e gerar **OpenAPI** a partir dos mesmos schemas. O pacote é publicado no GitHub Packages e consumido pelo `projeta-web` com versão exata. Erros seguem **RFC 9457 (Problem Details)**.

## Justificativa

- Uma única fonte para validação de runtime e tipo estático, mesmo com repositórios separados.
- OpenAPI deixa de ser documentação manual que envelhece.
- Workers e IA reutilizam os mesmos schemas de entrada/saída.

## Consequências

- Toda rota declara schema de body, query, params e resposta.
- Schemas de resposta são versionados; mudança incompatível exige `/v2`.
- Testes de contrato garantem que a resposta real valida no schema.
- Ferramenta de geração OpenAPI precisa suportar Zod (definida na FASE 0).

## Alternativas

- Copiar contratos nos dois repositórios: rejeitado por deriva silenciosa.
- class-validator + Swagger decorators: idiomático em NestJS, mas duplica tipo e validação.
- TypeSpec/OpenAPI-first: mais cerimônia; Zod atende melhor ao fluxo atual.
