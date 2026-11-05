# SPEC-001 — Identidade e autenticação

| Campo | Valor |
|:--|:--|
| ID | SPEC-001 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 |
| Dono | Backend |
| Depende de | SPEC-000 |
| Requisitos cobertos | RF-001, RF-002, RF-003, RF-004, RF-005, RB-003, RB-016 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Permitir que uma pessoa crie conta, autentique-se com segurança, recupere acesso, mantenha seus dados de conta atualizados e solicite exclusão, com consentimentos registrados e sessões revogáveis.

## 2. Fora de escopo

- Perfil profissional (SPEC-003).
- Login social/OAuth (não previsto no MVP).
- MFA (não previsto; pode virar decisão futura).
- Gestão de organizações (SPEC-002).

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-001 | cadastro com nome, e-mail, senha, aceite de termos e privacidade |
| RF-002 | login com e-mail e senha, sessão por cookie |
| RF-003 | recuperação de senha por e-mail com token de uso único |
| RF-004 | consulta e atualização dos dados de conta |
| RF-005 | solicitação de exclusão com retenção e anonimização |
| RB-016 | eventos de identidade auditados |

## 4. Regras e invariantes

- E-mail único, normalizado (`trim` + lowercase).
- Senha: mínimo 10 caracteres, máximo 128; bloqueio de senhas comuns; hash **Argon2id**.
- Mensagens de login e recuperação não revelam existência de conta.
- Access token JWT de 15 minutos; refresh de 30 dias, rotativo, revogável.
- Reutilização de refresh revoga toda a família de tokens.
- Cadastro exige aceite explícito das versões vigentes de termos e privacidade (versão + data + IP registrados).
- Conta `SUSPENDED` não autentica; `DELETED` não autentica e não aparece em buscas.
- Exclusão:
  - revoga todas as sessões;
  - falha se o usuário for `OWNER` de organização com outros membros (exige transferência);
  - assinatura ativa é cancelada ao fim do período vigente; registros financeiros são retidos pelo prazo legal;
  - marca `deletedAt`, inicia retenção legal e agenda anonimização.
- Nenhuma senha, token ou hash aparece em log.

## 5. Modelo de dados

### `users`

| Campo | Tipo | Regra |
|:--|:--|:--|
| `_id` | ObjectId | |
| `name` | string | 2..120 |
| `email` | string | único, normalizado |
| `passwordHash` | string | Argon2id, nunca exposto |
| `emailVerifiedAt` | Date? | |
| `status` | enum | `ACTIVE`, `SUSPENDED`, `DELETED` |
| `acceptedTermsVersion` | string | |
| `acceptedPrivacyVersion` | string | |
| `acceptedAt` | Date | |
| `consentIp` | string | |
| `locale` | string | default `pt-BR` |
| `timezone` | string | default `America/Manaus` |
| `createdAt` / `updatedAt` / `deletedAt` | Date | UTC |

### `user_sessions`

| Campo | Tipo | Regra |
|:--|:--|:--|
| `_id` | ObjectId | |
| `userId` | ObjectId | índice |
| `familyId` | string | rotação |
| `refreshHash` | string | SHA-256 do token |
| `userAgent` / `ip` | string | auditoria |
| `expiresAt` | Date | TTL index |
| `revokedAt` | Date? | |

### `email_tokens`

`{ _id, userId, type: 'VERIFY_EMAIL'|'RESET_PASSWORD', tokenHash, expiresAt, usedAt?, createdAt }`

## 6. Contratos de API

| Método | Rota | Auth | Request | Response |
|:--|:--|:--|:--|:--|
| POST | `/auth/register` | pública | `{ name, email, password, acceptTerms: true, acceptPrivacy: true, termsVersion, privacyVersion }` | `201 { userId }` |
| POST | `/auth/login` | pública | `{ email, password }` | `200` + cookies |
| POST | `/auth/logout` | sessão | — | `204` |
| POST | `/auth/refresh` | cookie | — | `200` + novos cookies |
| POST | `/auth/forgot-password` | pública | `{ email }` | `202` (sempre) |
| POST | `/auth/reset-password` | pública | `{ token, password }` | `204` |
| GET | `/users/me` | sessão | — | `{ id, name, email, locale, timezone, status, createdAt }` |
| PATCH | `/users/me` | sessão | `{ name?, locale?, timezone? }` | `200` usuário |
| POST | `/users/me/email/verify` | sessão | `{ token }` | `204` |
| POST | `/users/me/email/resend` | sessão | — | `202` |
| POST | `/users/me/password` | sessão | `{ currentPassword, newPassword }` | `204` |
| POST | `/users/me/sessions/revoke-all` | sessão | — | `204` |
| DELETE | `/users/me` | sessão | `{ password, reason? }` | `202` |

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 409 | `AUTH_EMAIL_TAKEN` | e-mail já cadastrado |
| 401 | `AUTH_INVALID_CREDENTIALS` | login inválido (genérico) |
| 403 | `AUTH_EMAIL_NOT_VERIFIED` | ação que exige verificação |
| 403 | `AUTH_ACCOUNT_SUSPENDED` | conta suspensa |
| 410 | `AUTH_TOKEN_EXPIRED` | token de e-mail expirado |
| 422 | `AUTH_WEAK_PASSWORD` | política de senha |
| 409 | `ACCOUNT_DELETE_BLOCKED` | owner de organização com outros membros |
| 429 | `RATE_LIMITED` | excesso de tentativas |

## 7. Segurança

- Rate limit: 5 tentativas de login por 15 min por IP+e-mail; 3 envios de e-mail por hora por conta.
- Cookies: `httpOnly`, `Secure`, `SameSite=Lax`, `Path=/`; `Domain` vazio em local e `.<dominio>` em produção/staging (ADR-023), para funcionar com web na Vercel e API na VPS.
- CSRF token exigido em `PATCH`/`POST`/`DELETE` (exceto login/registro públicos, protegidos por rate limit).
- Comparação de token em tempo constante.
- Token de e-mail: 32 bytes aleatórios, armazenado por hash, TTL 30 min.
- Logout revoga a sessão atual; `revoke-all` revoga todas as famílias.
- Auditoria: `user.registered`, `user.login`, `user.login_failed`, `user.password_changed`, `user.deleted`, `session.revoked`.

## 8. Critérios de aceite

```gherkin
Cenário: cadastro com e-mail duplicado
  Dado que existe uma conta com "ana@exemplo.com"
  Quando alguém tenta cadastrar o mesmo e-mail
  Então a resposta é 409 AUTH_EMAIL_TAKEN
  E nenhuma conta nova é criada

Cenário: login não revela existência de conta
  Dado um e-mail não cadastrado
  Quando a senha é enviada
  Então a resposta é 401 com a mesma mensagem de senha incorreta

Cenário: refresh rotativo detecta reutilização
  Dado um refresh token já utilizado
  Quando ele é reapresentado
  Então a resposta é 401
  E toda a família de sessões é revogada

Cenário: exclusão bloqueada para owner
  Dado um usuário OWNER de organização com 2 membros
  Quando solicita exclusão
  Então a resposta é 409 ACCOUNT_DELETE_BLOCKED
  E a conta permanece ativa

Cenário: recuperação sem enumeração
  Dado um e-mail inexistente
  Quando solicita recuperação
  Então a resposta é 202
  E nenhum e-mail é enviado
```

## 9. Testes

- **Unitários:** política de senha, normalização de e-mail, geração/validação de token, guards de status.
- **Integração:** registro/login/refresh/logout com Mongo e Redis; TTL de sessão; revogação em família; e-mail via Mailpit.
- **E2E:** fluxo completo de cadastro → verificação → login → logout → recuperação.
- **Segurança:** brute force, CSRF, fixation, enumeração, cookie flags, reutilização de refresh.

## 10. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-010 | MFA obrigatório para ADMIN? | médio | decidir antes do plano STUDIO |
| OPEN-011 | validade de access/refresh | baixo | calibrar em staging |
| ASM-002 | política de senha mínima de 10 caracteres | baixo | validar com Product |

## 11. Definition of Done específico

- [ ] Testes de segurança de autenticação verdes.
- [ ] Cookies e CSRF verificados em navegador real.
- [ ] E-mails transacionais em Mailpit/staging.
- [ ] Auditoria gravando eventos.
- [ ] Documentação OpenAPI atualizada.
- [ ] Nenhum segredo em log (teste automatizado).

## 12. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
