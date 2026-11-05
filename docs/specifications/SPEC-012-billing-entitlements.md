# SPEC-012 — Billing, planos e entitlements

| Campo | Valor |
|:--|:--|
| ID | SPEC-012 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 |
| Dono | Backend |
| Depende de | SPEC-001, SPEC-002 |
| Requisitos cobertos | RF-110..115, RB-012, RB-013, RB-017 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Monetizar o produto com planos claros, cobrança confiável e limites aplicados no servidor — sem punir o usuário com perda de dados e sem permitir acesso pago por manipulação do frontend.

## 2. Fora de escopo

- Emissão de nota fiscal (integração fiscal).
- Cobrança por projeto ou por sucesso.
- Marketplace de planos.
- Créditos pré-pagos (avaliar no futuro).

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-110 | planos FREE, INDIVIDUAL, PROFESSIONAL, STUDIO |
| RF-111 | contratar, trocar e cancelar plano |
| RF-112 | limites de projetos, análises, gerações, storage, usuários, exportações |
| RF-113 | webhooks com assinatura e idempotência |
| RF-114 | estados TRIAL, ACTIVE, PAST_DUE, CANCELLED, EXPIRED, SUSPENDED |
| RF-115 | autorização e limites validados no servidor |

## 4. Planos e entitlements

### 4.1 Catálogo

| Plano | Público | Recursos principais |
|:--|:--|:--|
| FREE | experimentação | consulta de oportunidades, perfil básico, projetos e análises limitados |
| INDIVIDUAL | artista/profissional | radar personalizado, assistência de IA, mais análises, exportação |
| PROFESSIONAL | produtor/consultor | mais análises e projetos, orçamento, cronograma, gestão documental |
| STUDIO | equipes/organizações | membros, colaboração, portfólio ampliado |

### 4.2 Métricas de quota

| Métrica | Escopo | Janela |
|:--|:--|:--|
| `projects.active` | tenant | simultâneo |
| `analyses.monthly` | tenant | mês calendário |
| `ai.generations.monthly` | tenant | mês calendário |
| `exports.monthly` | tenant | mês calendário |
| `storage.bytes` | tenant | simultâneo |
| `members.max` | organização | simultâneo |
| `opportunity.alerts` | tenant | simultâneo |

**Valores numéricos não são definidos aqui** (decisão `OPEN-025`). O código lê `plan.limits`; nenhum limite é hardcoded.

### 4.3 Features

`radar.personalized`, `ai.assistant`, `budget`, `schedule`, `documents.management`, `team.collaboration`, `exports.docx`, `exports.pdf`, `eligibility.advanced`.

## 5. Assinatura

### 5.1 Estados

```
TRIAL ──► ACTIVE ──► PAST_DUE ──► ACTIVE
             │            ├──► SUSPENDED ──► ACTIVE
             │            └──► CANCELLED ──► EXPIRED
             └──► CANCELLED ──► EXPIRED
```

### 5.2 Regras de acesso

| Status | Leitura | Escrita | Gerações pagas |
|:--|:--:|:--:|:--:|
| TRIAL | ✅ | ✅ | ✅ (limitado) |
| ACTIVE | ✅ | ✅ | ✅ |
| PAST_DUE | ✅ | ✅ | ❌ |
| SUSPENDED | ✅ | ❌ | ❌ |
| CANCELLED | ✅ até fim do período | ✅ até fim do período | ✅ até fim do período |
| EXPIRED | ✅ (plano FREE) | conforme FREE | conforme FREE |

- `CANCELLED` não apaga projetos (RB-013).
- `PAST_DUE` nunca bloqueia leitura ou exportação do que já é do usuário.

### 5.3 Trial

- Duração configurável (`OPEN-042`).
- Um trial por tenant; abuso detectado bloqueia novo trial.
- Fim do trial sem conversão → FREE.

## 6. Entitlements e quota (server-side)

### 6.1 Verificação

```
1. resolver tenant (user/org) e assinatura vigente
2. se sem assinatura → plano FREE
3. carregar plan.version vigente no período
4. feature flag? se ausente → 402 ENTITLEMENT_REQUIRED
5. quota: reservar atomicamente (Redis INCR + limite)
6. executar operação
7. confirmar consumo em usage_records; falha → liberar reserva
```

- Redis é a fonte de verdade do contador do período; `usage_records` é o registro durável e reconciliável.
- Job diário reconcilia contadores (Redis × banco) e corrige divergência.
- Nenhum endpoint confia em `plan` ou `quota` enviados pelo cliente.
- O frontend apenas oculta ações; a API sempre valida.

### 6.2 Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 402 | `ENTITLEMENT_REQUIRED` | feature não incluída no plano |
| 429 | `QUOTA_EXCEEDED` | limite do período atingido |
| 409 | `SUBSCRIPTION_STATE_INVALID` | ação incompatível com status |
| 422 | `PLAN_NOT_AVAILABLE` | plano inativo/inexistente |
| 409 | `TRIAL_ALREADY_USED` | trial repetido |

## 7. Modelo de dados

### `plans`

`{ _id, code, name, price, currency, interval: MONTHLY|YEARLY, limits, features, version, active, createdAt }`

### `subscriptions`

`{ _id, tenantKey, planCode, status, currentPeriodStart, currentPeriodEnd, trialEndsAt?, cancelAt?, cancelledAt?, provider, providerCustomerId, providerSubscriptionId, createdAt, updatedAt }`

### `usage_records`

`{ _id, tenantKey, metric, period: 'YYYY-MM', used, reserved, updatedAt }` — único por `(tenantKey, metric, period)`.

### `billing_events` (derivada — RF-113)

`{ _id, provider, providerEventId, type, payloadHash, processedAt, status }` — único por `providerEventId`.

> Coleção adicional necessária para idempotência de webhook; registrada no relatório de consistência.

## 8. Contratos de API

| Método | Rota | Permissão | Request | Response |
|:--|:--|:--|:--|:--|
| GET | `/billing/plans` | pública | — | catálogo ativo |
| GET | `/billing/subscription` | autenticado | — | assinatura + uso |
| POST | `/billing/checkout` | OWNER | `{ planCode, interval }` | `200 { checkoutUrl }` |
| POST | `/billing/cancel` | OWNER | `{ atPeriodEnd: true }` | `200` |
| POST | `/billing/webhooks/{provider}` | assinatura do provedor | payload bruto | `200` sempre após validar |
| GET | `/billing/usage` | autenticado | — | quotas e consumo |

### Webhook

- Validar assinatura HMAC e timestamp (tolerância de 5 min).
- Persistir evento bruto antes de processar.
- `providerEventId` único; duplicata responde `200` sem reprocessar.
- Processamento assíncrono (`billing-webhooks`), resposta rápida.
- Eventos fora de ordem: ignorar evento com timestamp anterior ao último aplicado para a mesma assinatura.

## 9. Segurança e fraude

- Preço sempre do catálogo no servidor; nunca do cliente.
- Checkout vinculado ao tenant autenticado.
- Webhook não autenticado por sessão; somente por assinatura do provedor.
- Log de eventos financeiros sem dados de cartão.
- Limite de tentativas de checkout.

## 10. Critérios de aceite

```gherkin
Cenário: limite é aplicado no servidor
  Dado um tenant no plano FREE com limite de análises atingido
  Quando ele chama POST /edital-analyses
  Então a resposta é 429 QUOTA_EXCEEDED
  E nenhum job é criado

Cenário: frontend não concede acesso
  Dado um usuário FREE que adultera o estado local para "PROFESSIONAL"
  Quando chama um endpoint da feature budget
  Então a resposta é 402 ENTITLEMENT_REQUIRED

Cenário: webhook duplicado não duplica efeito
  Dado o mesmo providerEventId recebido duas vezes
  Quando o segundo é processado
  Então a resposta é 200
  E a assinatura não muda

Cenário: cancelamento preserva projetos
  Dado um tenant com projetos ativos
  Quando a assinatura é cancelada e expira
  Então os projetos continuam acessíveis para leitura
  E nenhum projeto é excluído

Cenário: quota é liberada em falha
  Dado uma geração que falha no provedor
  Quando o job termina em erro
  Então a reserva de quota é liberada
```

## 11. Testes

- **Unitários:** cálculo de período, transições de assinatura, avaliador de feature/quota.
- **Integração:** checkout fake, webhook assinado/duplicado/fora de ordem, reconciliação.
- **E2E:** assinar, usar limite, atingir quota, cancelar.
- **Segurança:** preço manipulado, webhook sem assinatura, replay, acesso cruzado de tenant.

## 12. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-001 | provedor de pagamento | alto | ADR antes da Fase 1 |
| OPEN-025 | limites e preços por plano | alto | decisão comercial |
| OPEN-042 | duração do trial e regra de cancelamento | médio | decisão comercial |
| OPEN-043 | cobrança por assento no STUDIO | médio | decidir antes do STUDIO |

## 13. Definition of Done específico

- [ ] Nenhum limite aplicado apenas no frontend.
- [ ] Webhook idempotente e verificado.
- [ ] Quota liberada em falha.
- [ ] Reconciliação Redis × banco.
- [ ] Cancelamento não apaga dados.

## 14. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
