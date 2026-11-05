# SPEC-002 — Organizações, membros e tenancy

| Campo | Valor |
|:--|:--|
| ID | SPEC-002 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 |
| Dono | Backend |
| Depende de | SPEC-000, SPEC-001 |
| Requisitos cobertos | RF-010..013, RB-005, RB-006, RB-007, RB-016 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Permitir que profissionais atuem individualmente ou por organizações, com papéis claros, convites, transferência de propriedade e isolamento total de dados entre tenants.

## 2. Fora de escopo

- Cobrança por assento (SPEC-012 trata limites; preço por membro é decisão aberta).
- SSO corporativo.
- Estrutura hierárquica de organizações (matriz/filial).

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-010 | cadastro de organização com dados cadastrais e área de atuação |
| RF-011 | convite e associação de usuários |
| RF-012 | papéis OWNER, ADMIN, EDITOR, VIEWER |
| RF-013 | isolamento: sem membership ativa, sem acesso |
| RB-005 | todo recurso privado tem proprietário |
| RB-006/007 | autorização no backend; sem IDOR |

## 4. Matriz de capacidades

| Capacidade | VIEWER | EDITOR | ADMIN | OWNER |
|:--|:--:|:--:|:--:|:--:|
| Ver organização e membros | ✅ | ✅ | ✅ | ✅ |
| Ver projetos/documentos da organização | ✅ | ✅ | ✅ | ✅ |
| Criar/editar projeto | ❌ | ✅ | ✅ | ✅ |
| Excluir/arquivar projeto | ❌ | ❌ | ✅ | ✅ |
| Exportar proposta/documento | ❌ | ✅ | ✅ | ✅ |
| Convidar/remover membros | ❌ | ❌ | ✅ | ✅ |
| Alterar papel de membro | ❌ | ❌ | ✅ | ✅ |
| Editar dados da organização | ❌ | ❌ | ✅ | ✅ |
| Gerenciar assinatura | ❌ | ❌ | ❌ | ✅ |
| Transferir propriedade | ❌ | ❌ | ❌ | ✅ |
| Arquivar organização | ❌ | ❌ | ❌ | ✅ |

Regra: **ADMIN não pode promover a OWNER nem remover o OWNER**.

## 5. Invariantes

- Exatamente **um** `OWNER` ativo por organização.
- `(organizationId, userId)` único em `memberships`.
- `INVITED` não concede acesso a dados.
- Remover o último `OWNER` é proibido; transferência é obrigatória.
- `cnpj`, quando presente, é único e válido.
- Organização arquivada não aceita novos membros, projetos ou cobrança.
- Nenhum endpoint aceita `organizationId` como fonte de autorização; o vínculo é sempre verificado.

## 6. Modelo de dados

### `organizations`

`{ _id, legalName, tradeName?, cnpj?, legalNature?, address { line1, line2?, city, state, postalCode, country }, areas[], responsibleUserId, status: ACTIVE|ARCHIVED, createdAt, updatedAt }`

### `memberships`

`{ _id, organizationId, userId, role, status: INVITED|ACTIVE|SUSPENDED|REMOVED, invitedBy?, invitedAt?, acceptedAt?, createdAt, updatedAt }`

### `membership_invites` (derivada)

`{ _id, organizationId, email, role, tokenHash, invitedBy, expiresAt, acceptedAt? }`

## 7. Contratos de API

| Método | Rota | Permissão | Request | Response |
|:--|:--|:--|:--|:--|
| POST | `/organizations` | autenticado | `{ legalName, tradeName?, cnpj?, legalNature?, address, areas[] }` | `201` org (criador vira OWNER) |
| GET | `/organizations` | autenticado | — | organizações em que é membro ativo |
| GET | `/organizations/{organizationId}` | VIEWER+ | — | org + papel do solicitante |
| PATCH | `/organizations/{organizationId}` | ADMIN+ | campos editáveis | `200` |
| POST | `/organizations/{organizationId}/members` | ADMIN+ | `{ email, role }` | `201` convite |
| PATCH | `/organizations/{organizationId}/members/{userId}` | ADMIN+ | `{ role }` | `200` |
| DELETE | `/organizations/{organizationId}/members/{userId}` | ADMIN+ | — | `204` |
| POST | `/organizations/{organizationId}/transfer-ownership` | OWNER | `{ userId }` | `200` |
| POST | `/organizations/{organizationId}/invites/accept` | autenticado | `{ token }` | `200` membership |
| POST | `/organizations/{organizationId}/archive` | OWNER | `{ confirm: true }` | `200` |

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 403 | `TENANT_FORBIDDEN` | sem membership ou membership suspensa |
| 403 | `ROLE_FORBIDDEN` | papel insuficiente |
| 409 | `ORG_ALREADY_EXISTS` | CNPJ duplicado |
| 409 | `OWNER_TRANSFER_REQUIRED` | remoção do último owner |
| 409 | `MEMBER_ALREADY_EXISTS` | convite duplicado |
| 410 | `INVITE_EXPIRED` | convite vencido |
| 422 | `INVALID_CNPJ` | dígitos inválidos |

## 8. Convites

- Token de 32 bytes, hash no banco, validade de 7 dias.
- E-mail de convite contém link com token de uso único.
- Aceite exige que o e-mail autenticado seja o mesmo do convite.
- Reenvio invalida o token anterior.
- Convite para e-mail já membro retorna `409`.

## 9. Tenancy

- `TenantContext = { ownerType, ownerId, tenantKey, role }`.
- Rotas com `{organizationId}`: guard valida membership ativa e papel mínimo.
- Rotas de recurso (`/projects/{id}`): guard carrega o recurso e valida o `owner`; nunca confia em query/body.
- Repositórios recebem `TenantContext` obrigatório (ADR-015).
- Teste de arquitetura: nenhuma rota privada sem `@RequireTenant()`.
- Teste de IDOR: matriz de acesso cruzado entre dois tenants.

## 10. Critérios de aceite

```gherkin
Cenário: convite aceito por usuário correto
  Dado um convite para "bia@exemplo.com" como EDITOR
  Quando Bia aceita autenticada com esse e-mail
  Então ela passa a ser membro ativo
  E pode criar projetos na organização

Cenário: convite para outro e-mail é recusado
  Dado o mesmo convite
  Quando outro usuário tenta aceitar
  Então a resposta é 403
  E nenhum vínculo é criado

Cenário: VIEWER não cria projeto
  Dado um membro VIEWER
  Quando ele tenta POST /projects no tenant da organização
  Então a resposta é 403 ROLE_FORBIDDEN

Cenário: último owner não pode ser removido
  Dado que a organização tem um único OWNER
  Quando um ADMIN tenta removê-lo
  Então a resposta é 409 OWNER_TRANSFER_REQUIRED
```

## 11. Testes

- **Unitários:** matriz de capacidades, validação de CNPJ, transições de membership.
- **Integração:** convite → aceite → acesso; transferência de ownership; arquivamento.
- **E2E:** criar organização, convidar, aceitar, criar projeto, remover membro.
- **Segurança:** IDOR entre organizações; escalada de papel; convite forjado; acesso com membership suspensa.

## 12. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-007 | transferir projeto de usuário para organização | médio | definir regra e auditoria |
| OPEN-012 | limite de membros por plano | médio | depende de SPEC-012 |
| OPEN-013 | organizações com múltiplos CNPJ | baixo | fora do MVP |

## 13. Definition of Done específico

- [ ] Matriz de capacidades coberta por testes.
- [ ] Nenhuma rota de organização sem guard.
- [ ] Teste de IDOR cross-tenant verde.
- [ ] Auditoria de mudanças de papel e ownership.
- [ ] Convite e aceite testados com e-mail real em staging.

## 14. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
