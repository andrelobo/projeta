# SPEC-011 — Dashboard, alertas e notificações

| Campo | Valor |
|:--|:--|
| ID | SPEC-011 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 |
| Dono | Backend / Frontend |
| Depende de | SPEC-004, SPEC-008 |
| Requisitos cobertos | RF-037, RF-100..103 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Dar ao usuário uma visão acionável do que importa agora: prazos, oportunidades relevantes, projetos em andamento, pendências e atividade recente — com alertas que não geram ruído.

## 2. Fora de escopo

- Analytics avançado e relatórios gerenciais (Fase 4).
- Recomendação por ML.
- Push mobile nativo.
- Dashboard de organização com agregações financeiras.

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-037 | alertas de oportunidades relevantes |
| RF-100 | dashboard com oportunidades, prazos, projetos, pendências e atividade |
| RF-101 | projetos por status |
| RF-102 | tarefas vinculadas a projetos |
| RF-103 | alertas de vencimento |

## 4. Dashboard

### 4.1 Blocos

| Bloco | Conteúdo | Fonte |
|:--|:--|:--|
| Oportunidades relevantes | até 6, por match de área/UF/tipo + favoritas | `opportunities` + perfil |
| Prazos próximos | encerramentos em 7/30 dias (favoritas e salvas) | `opportunities` |
| Projetos em andamento | por status, ordenados por atualização | `projects` |
| Submetidos | propostas `SUBMITTED` e resultado registrado | `project_opportunities` |
| Pendências | seções vazias, documentos faltantes, elegibilidade incompleta, análise não revisada | agregado |
| Atividade recente | últimas 10 ações do usuário/tenant | `audit_logs` (visão resumida) |

### 4.2 Regras de negócio

- "Relevante" usa match determinístico: área em comum, UF compatível, tipo de proponente elegível, prazo futuro, `effectiveStatus = OPEN`.
- Nunca exibir oportunidade `CLOSED` como prazo.
- Pendências são derivadas, não persistidas; no máximo cacheadas por 60 s.
- Dashboard de organização agrega somente recursos do tenant.
- Contagens respeitam o plano (não mostrar recurso bloqueado como disponível).

## 5. Tarefas (RF-102)

No MVP, tarefa é uma `project_activities` simplificada:

```ts
{ _id, projectId, proposalId?, kind: 'TASK', title, dueDate?, status: 'OPEN'|'DONE', assignedTo?, createdAt }
```

- Não possui dependências nem etapas no MVP (Fase 2 adiciona).
- Concluir tarefa é idempotente.
- Tarefa vencida aparece em "Pendências" e dispara notificação.

## 6. Notificações

### 6.1 Tipos

| Tipo | Gatilho |
|:--|:--|
| `OPPORTUNITY_MATCH` | busca salva encontrou nova oportunidade |
| `DEADLINE_APPROACHING` | 7, 3 e 1 dia antes do encerramento |
| `ANALYSIS_READY` | análise de edital concluída |
| `ANALYSIS_FAILED` | falha na análise |
| `ELIGIBILITY_READY` | avaliação concluída |
| `EXPORT_READY` | documento gerado |
| `PROPOSAL_SUBMITTED` | submissão registrada |
| `QUOTA_WARNING` | 80% e 100% do limite |
| `SUBSCRIPTION_CHANGED` | pagamento/assinatura |
| `TASK_DUE` | tarefa vencendo |

### 6.2 Canais

`IN_APP` (sempre), `EMAIL` (preferência do usuário). Push fica para Fase 4.

### 6.3 Regras

- Idempotência por `(eventId, channel)`.
- Preferências por tipo e canal; padrão: in-app ligado, e-mail ligado para prazos e cobrança.
- Agrupamento diário para oportunidades (evitar 20 e-mails).
- Notificação nunca contém conteúdo de outro tenant nem dado sensível desnecessário.
- Marcar como lida é idempotente; "marcar todas" também.

## 7. Alertas por busca salva

- `saved_searches` (SPEC-004) define filtros e frequência.
- Job `opportunity.alerts` roda diariamente/semanalmente.
- Para cada busca: encontra oportunidades novas desde `lastRunAt` que casam com os filtros e não foram notificadas.
- Cria uma notificação por oportunidade (in-app) e um resumo por e-mail.
- `lastNotifiedAt` evita repetição.

## 8. Modelo de dados

### `notifications`

`{ _id, userId, tenantKey, type, title, body, payload, readAt?, channelsSent: ['IN_APP','EMAIL'], createdAt }`

### `notification_preferences`

`{ _id, userId, preferences: { [type]: { inApp: boolean, email: boolean } } }`

### `notification_deliveries` (derivada)

`{ _id, notificationId, channel, status: QUEUED|SENT|FAILED, attempt, providerMessageId?, createdAt }`

## 9. Contratos de API

| Método | Rota | Permissão | Descrição |
|:--|:--|:--|:--|
| GET | `/dashboard/summary` | autenticado | blocos agregados |
| GET | `/dashboard/deadlines?days=7` | autenticado | prazos |
| GET | `/dashboard/recent-activity` | autenticado | últimas ações |
| GET | `/notifications?unread=true` | autenticado | lista paginada |
| PATCH | `/notifications/{notificationId}/read` | dono | marca lida |
| POST | `/notifications/read-all` | autenticado | marca todas |
| GET | `/notifications/preferences` · PATCH | autenticado | preferências |
| POST | `/tasks` | autenticado | cria tarefa |
| PATCH | `/tasks/{taskId}` | dono/EDITOR+ | atualiza (inclusive `DONE`) |
| GET | `/projects/{projectId}/tasks` | VIEWER+ | lista |

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 404 | `NOTIFICATION_NOT_FOUND` | id inexistente/sem acesso |
| 422 | `TASK_PROJECT_REQUIRED` | tarefa sem projeto |
| 403 | `TASK_FORBIDDEN` | VIEWER tentando editar |

## 10. Critérios de aceite

```gherkin
Cenário: prazo não mostra oportunidade encerrada
  Dado uma oportunidade favorita cujo prazo passou
  Quando o dashboard é carregado
  Então ela não aparece em "prazos próximos"

Cenário: alerta não duplica
  Dado uma busca salva diária já notificada hoje
  Quando o job roda novamente no mesmo dia
  Então nenhuma nova notificação é criada para a mesma oportunidade

Cenário: notificação isolada por usuário
  Dado notificações de A e B
  Quando A lista suas notificações
  Então nenhuma de B aparece

Cenário: tarefa vencida entra em pendências
  Dado uma tarefa OPEN com dueDate no passado
  Quando o dashboard é carregado
  Então ela aparece em pendências
  E gera notificação TASK_DUE
```

## 11. Testes

- **Unitários:** match de relevância, cálculo de pendências, dedupe de alerta.
- **Integração:** job de alertas, envio de e-mail (Mailpit), preferências.
- **E2E:** dashboard, marcar notificação, criar tarefa, concluir.
- **Segurança:** notificações por tenant; payload sem dado sensível.

## 12. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-039 | provedor de e-mail | médio | ADR |
| OPEN-040 | agrupamento e frequência de e-mail | baixo | calibrar com uso |
| OPEN-041 | notificações por organização | médio | definir com Product |

## 13. Definition of Done específico

- [ ] Dashboard não exibe item encerrado como aberto.
- [ ] Alertas idempotentes.
- [ ] Preferências respeitadas.
- [ ] Nenhum vazamento entre usuários/tenants.
- [ ] E-mail em staging validado.

## 14. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
