# ADR-009 — Multi-tenancy desde a fundação

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

O plano STUDIO prevê organizações e equipes. Adicionar tenancy depois exigiria migrar dados, reescrever autorização e refazer índices.

## Decisão

Adotar **multi-tenancy desde o primeiro commit**: todo recurso privado tem `ownerType/ownerId`, `tenantKey` e autorização centralizada por `TenantGuard`.

## Consequências

- Nenhum recurso privado existe sem proprietário.
- Consultas sempre filtram por tenant; testes de IDOR são obrigatórios.
- Usuário individual é um tenant (`user:{id}`), o que uniformiza quotas e auditoria.
- Complexidade inicial maior, mas sem migração dolorosa depois.

## Alternativas consideradas

- Single-tenant no MVP com migração futura: rejeitado por custo de reescrita.
