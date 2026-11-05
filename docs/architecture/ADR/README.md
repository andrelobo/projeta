# ADRs — Índice

Decisões arquiteturais do PROJETA. ADRs **Aceitos** vieram do PRD 1.0. ADRs **Propostos** foram derivados nesta fase de planejamento e **requerem aprovação humana** antes da implementação.

| ID | Título | Status | Impacto |
|:--|:--|:--|:--|
| [ADR-001](ADR-001-monolito-modular.md) | Monólito modular NestJS | Aceito | Estrutura da API |
| [ADR-002](ADR-002-nextjs-frontend.md) | Next.js no frontend | Aceito | Stack web |
| [ADR-003](ADR-003-mongodb-atlas.md) | MongoDB Atlas | Aceito | Persistência |
| [ADR-004](ADR-004-redis-bullmq.md) | Redis + BullMQ | Aceito | Assíncrono |
| [ADR-005](ADR-005-llm-desacoplado.md) | LLM desacoplado | Aceito | Camada AI |
| [ADR-006](ADR-006-culture-primeiro.md) | CULTURE primeiro | Aceito | Escopo |
| [ADR-007](ADR-007-health-isolado.md) | HEALTH isolado | Aceito | Domínios |
| [ADR-008](ADR-008-fontes-oficiais-autoridade.md) | Fontes oficiais | Aceito | Confiabilidade |
| [ADR-009](ADR-009-multi-tenancy-fundacao.md) | Multi-tenancy na fundação | Aceito | Autorização |
| [ADR-010](ADR-010-sdd-obrigatorio.md) | SDD obrigatório | Aceito | Processo |
| [ADR-011](ADR-011-monorepo-pnpm.md) | Monorepo pnpm | **Rejeitado** | Tooling |
| [ADR-012](ADR-012-contratos-zod-openapi.md) | Zod + OpenAPI + Problem Details | Aceito | Contratos |
| [ADR-013](ADR-013-money-decimal128.md) | Money/Decimal128 | Aceito | Dinheiro |
| [ADR-014](ADR-014-auth-cookie-csrf.md) | Cookie httpOnly + CSRF | Aceito | Autenticação |
| [ADR-015](ADR-015-ownership-tenantkey.md) | OwnerRef + tenantKey | Aceito | Tenancy |
| [ADR-016](ADR-016-status-efetivo-oportunidade.md) | Status efetivo + verificação | Aceito | Radar |
| [ADR-017](ADR-017-proposta-versao.md) | Proposta × versão | Aceito | Projetos |
| [ADR-018](ADR-018-storage-s3-assinado.md) | Storage S3 assinado | Aceito | Documentos |
| [ADR-019](ADR-019-jobs-idempotencia-outbox.md) | Idempotência + outbox + DLQ | Aceito | Filas |
| [ADR-020](ADR-020-prompt-registry-custo.md) | Prompt registry + custo | Aceito | IA |
| [ADR-021](ADR-021-polyrepo.md) | Polyrepo: docs, web e API | Aceito | Repositórios |
| [ADR-022](ADR-022-deploy-vercel-vps.md) | Deploy Vercel + VPS (topologia) | Aceito | Infraestrutura |
| [ADR-023](ADR-023-auth-cross-domain.md) | Auth Vercel ↔ VPS | Aceito | Autenticação |
| [ADR-024](ADR-024-deploy-github-actions.md) | Deploy via GitHub Actions | Aceito | Deploy |
| [ADR-025](ADR-025-licenca-apache-2.md) | Licença Apache-2.0 | Aceito | Legal |

## Como propor um novo ADR

1. Copie a estrutura de um ADR existente.
2. Use `Status: Proposto`.
3. Descreva contexto, decisão, consequências e alternativas.
4. Registre no índice acima.
5. Peça aprovação humana antes de implementar.

## Como alterar um ADR aceito

Não edite silenciosamente. Crie um novo ADR que **supersede** o anterior e atualize o status do antigo para `Substituído por ADR-xxx`.
