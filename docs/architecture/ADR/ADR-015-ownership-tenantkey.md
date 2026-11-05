# ADR-015 — OwnerRef e tenantKey em todo recurso privado

- **Status:** Proposto — requer aprovação
- **Data:** 2026-11-05
- **Decisores:** System Architect
- **Relacionado:** ADR-009, RB-005, RB-007

## Contexto

O PRD exige isolamento por usuário/organização. Comparar `organizationId` do body é inseguro e gera IDOR.

## Decisão

Todo recurso privado armazena `owner { ownerType: USER|ORGANIZATION, ownerId }` e `tenantKey` (`user:{id}` / `org:{id}`). Repositórios exigem `TenantContext` não opcional. O guard resolve o proprietário a partir do recurso, nunca do cliente.

## Justificativa

- Uniformiza autorização, quotas, auditoria e índices.
- Impede esquecimento de filtro em consultas.
- Facilita migração futura para organização sem reescrever entidades.

## Consequências

- Escritas duplicam `tenantKey` (denormalização controlada).
- Teste automatizado verifica que todo repositório privado exige contexto.
- Mudança de propriedade (ex.: transferir projeto para organização) exige operação auditada e regra própria (`OPEN-007`).

## Alternativas

- Apenas `organizationId` opcional: rejeitado por ambiguidade e risco de IDOR.
- Row-level security no banco: MongoDB não oferece equivalente nativo.
