# SPEC-013 — Administração, fontes e auditoria

| Campo | Valor |
|:--|:--|
| ID | SPEC-013 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 |
| Dono | Backend |
| Depende de | SPEC-001, SPEC-004, SPEC-012 |
| Requisitos cobertos | RF-120..125, RB-016 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Dar à operação do PROJETA ferramentas para curar oportunidades, administrar fontes, acompanhar falhas e auditar ações — sem que administração se confunda com papel de organização.

## 2. Fora de escopo

- Impersonação de usuário (avaliar com cautela; `OPEN-044`).
- Backoffice financeiro completo (reembolso, disputa).
- Edição manual de conteúdo de proposta do usuário.

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-120 | consultar e administrar usuários conforme permissão |
| RF-121 | criar, revisar, verificar e arquivar oportunidades |
| RF-122 | administrar fontes |
| RF-123 | configurar planos e limites |
| RF-124 | consultar auditoria |
| RF-125 | visualizar falhas de processamento, jobs, análises e integrações |
| RB-016 | toda ação administrativa auditada |

## 4. Papéis de plataforma

Separados dos papéis de organização:

| Papel | Pode |
|:--|:--|
| `PLATFORM_ADMIN` | tudo em administração, exceto alterar próprio papel |
| `PLATFORM_OPERATOR` | curadoria de oportunidades e fontes; sem billing/usuários |
| `PLATFORM_SUPPORT` | leitura de usuários, assinaturas e jobs; sem escrita |
| `PLATFORM_AUDITOR` | leitura de auditoria e observabilidade |

Regra: papéis de plataforma são concedidos por `PLATFORM_ADMIN` e auditados; nunca herdados de organização.

## 5. Fontes de oportunidade

### 5.1 Interface de conector (PRD §15)

```ts
interface OpportunitySourceAdapter {
  discover(params: DiscoverParams): Promise<RawOpportunity[]>;
  fetchDetails(raw: RawOpportunity): Promise<RawOpportunityDetail>;
  normalize(detail: RawOpportunityDetail): Promise<NormalizedOpportunity>;
  verify(opportunity: Opportunity): Promise<VerificationResult>;
  healthCheck(): Promise<SourceHealth>;
}
```

Regras:

- `allowed = false` → nenhuma execução automática; apenas curadoria manual.
- Todo conector declara método de coleta, frequência, política de acesso e limitações.
- Falha repetida (`failureCount >= N`) pausa a fonte e alerta.
- Conector nunca publica direto: produz `DISCOVERED`/`PENDING` para revisão.
- Respeitar termos de uso e robots; sem contorno de proteção.

### 5.2 Administração de fontes

CRUD de `opportunity_sources`, com `lastRunAt`, `status`, `failureCount`, histórico de execuções e botão de teste (`healthCheck`).

## 6. Usuários e assinaturas

- Buscar por e-mail/nome; ver status, tenant, plano, uso.
- Suspender/reativar conta (com motivo e auditoria).
- Forçar logout (revoga sessões).
- Não é permitido ler documentos ou conteúdo de proposta do usuário sem base legal e registro.
- Ajuste manual de plano/limite: permitido, temporário, auditado, com expiração.

## 7. Planos

- CRUD de `plans` com versionamento: alteração cria nova versão; assinaturas existentes mantêm a versão do período.
- Ativar/desativar plano.
- Nenhuma alteração retroage cobrança já efetuada.

## 8. Auditoria

### 8.1 Eventos mínimos

`user.*`, `org.*`, `membership.*`, `opportunity.*`, `analysis.*`, `eligibility.*`, `project.*`, `version.*`, `proposal.*`, `document.*`, `billing.*`, `admin.*`, `ai.prompt_activated`.

### 8.2 Consulta

Filtros: período, ator, tenant, recurso, ação, resultado. Paginação por cursor. Exportação CSV auditada.

### 8.3 Retenção

Configurável (`OPEN-009`); mínimo de 12 meses para ações administrativas e financeiras (proposta).

## 9. Observabilidade operacional

Painel com:

- filas: profundidade, idade, falhas, DLQ;
- jobs: sucesso/falha por tipo e duração;
- análises: taxa de falha, tempo médio, custo médio;
- IA: custo por tenant, taxa de schema inválido, fallback;
- integrações: status por fonte e última execução;
- erros 5xx por rota.

Ações: reprocessar job, pausar fonte, reativar DLQ (com auditoria).

## 10. Contratos de API

| Método | Rota | Permissão | Descrição |
|:--|:--|:--|:--|
| GET | `/admin/users?q=` | SUPPORT+ | busca |
| POST | `/admin/users/{userId}/suspend` | ADMIN | suspende com motivo |
| POST | `/admin/users/{userId}/reactivate` | ADMIN | reativa |
| POST | `/admin/users/{userId}/revoke-sessions` | ADMIN | força logout |
| GET | `/admin/sources` · POST · PATCH | OPERATOR+ | fontes |
| POST | `/admin/sources/{sourceId}/test` | OPERATOR+ | healthCheck |
| POST | `/admin/sources/{sourceId}/run` | OPERATOR+ | executa discovery manual |
| GET | `/admin/plans` · POST · PATCH | ADMIN | planos versionados |
| GET | `/admin/audit` | AUDITOR+ | consulta |
| GET | `/admin/jobs?status=failed` | SUPPORT+ | jobs |
| POST | `/admin/jobs/{jobId}/retry` | OPERATOR+ | reprocessa |
| GET | `/admin/dlq` | OPERATOR+ | fila morta |
| POST | `/admin/dlq/{messageId}/retry` | OPERATOR+ | reprocessa mensagem |
| GET | `/admin/observability/summary` | SUPPORT+ | painel |

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 403 | `PLATFORM_ROLE_FORBIDDEN` | papel insuficiente |
| 409 | `SOURCE_NOT_ALLOWED` | execução com `allowed=false` |
| 422 | `ADMIN_REASON_REQUIRED` | ação sensível sem motivo |
| 409 | `LAST_PLATFORM_ADMIN` | remover o último admin |
| 404 | `ADMIN_RESOURCE_NOT_FOUND` | id inexistente |

## 11. Critérios de aceite

```gherkin
Cenário: operador não altera plano
  Dado um usuário PLATFORM_OPERATOR
  Quando ele tenta PATCH /admin/plans/{id}
  Então a resposta é 403 PLATFORM_ROLE_FORBIDDEN

Cenário: fonte desabilitada não executa
  Dado uma fonte com allowed=false
  Quando o job de descoberta roda
  Então nenhuma coleta é feita
  E um evento de bloqueio é registrado

Cenário: ação administrativa é auditada
  Dado um PLATFORM_ADMIN que suspende um usuário
  Quando a ação conclui
  Então audit_logs contém ator, motivo, antes/depois e requestId

Cenário: reprocessar job não duplica efeito
  Dado um job de exportação já concluído
  Quando o operador reenvia
  Então a chave de idempotência impede novo arquivo
```

## 12. Testes

- **Unitários:** matriz de papéis de plataforma, validação de motivo, versionamento de plano.
- **Integração:** curadoria, fonte bloqueada, retry de DLQ, auditoria.
- **E2E:** admin publica oportunidade, verifica, reprocessa job.
- **Segurança:** escalada para PLATFORM_ADMIN, acesso sem papel, auditoria imutável.

## 13. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-044 | impersonação de usuário | alto | decidir com jurídico |
| OPEN-045 | retenção de auditoria | médio | LGPD |
| OPEN-046 | acesso administrativo por IP/VPN | médio | decidir antes de produção |

## 14. Definition of Done específico

- [ ] Nenhuma rota admin acessível por papel de organização.
- [ ] Toda escrita admin auditada com motivo.
- [ ] Fonte bloqueada comprovadamente não executa.
- [ ] DLQ inspecionável e reprocessável.
- [ ] Painel operacional sem dados sensíveis.

## 15. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
