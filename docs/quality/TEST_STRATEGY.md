# Estratégia de testes — PROJETA

**Versão:** 1.0 (proposta) · **Status:** aguardando aprovação · **Base:** PRD §19, §23, RNF-004..006

> Regra central: **nenhuma funcionalidade é considerada concluída sem teste que prove o comportamento e a autorização.** Screenshot ou demonstração visual não são evidência suficiente (PRD §18).

---

## 1. Pirâmide e distribuição

| Camada | Proporção alvo | O que cobre | Ferramenta |
|:--|--:|:--|:--|
| Unitário | ~60% | regras de domínio, cálculos, máquinas de estado, validadores | Vitest |
| Integração | ~30% | API + Mongo + Redis + storage + fila + provider fake | Vitest + Supertest + Testcontainers |
| E2E | ~10% | jornadas críticas no navegador | Playwright |
| Segurança | transversal | IDOR, injeção, upload, webhook, prompt injection | Vitest + Playwright + scripts |
| Carga | sob demanda | p95, filas, custo | k6 (Fase 2) |
| Avaliação de IA | por tarefa | schema, groundedness, custo | harness próprio |

Cobertura não é meta isolada. Metas mínimas:

- **domínio crítico** (elegibilidade, dinheiro, versionamento, quota): ≥ 90% de branches;
- **global**: ≥ 75% de linhas;
- **rotas**: 100% com teste de autorização (positivo e negativo).

---

## 2. Ferramentas e convenções

| Item | Decisão |
|:--|:--|
| Runner | Vitest |
| HTTP | Supertest |
| Banco/Redis/storage | Testcontainers (Mongo, Redis, MinIO) |
| E2E | Playwright |
| Carga | k6 (Fase 2) |
| Contract | validação de resposta contra schema Zod/OpenAPI |
| Fixtures | factories determinísticas; sem dado real de cliente |
| Tempo | relógio injetável; nunca `sleep` em teste |
| Flaky | quarentena imediata; não faz merge com teste instável |

---

## 3. Ambientes

| Ambiente | Uso | Dados |
|:--|:--|:--|
| local | desenvolvimento | docker compose; seed sintético |
| CI | PR e main | efêmero; containers |
| staging | homologação e E2E | sintético; provedores fake ou sandbox |
| produção | smoke tests somente leitura | dados reais |

**Proibido:** usar dados reais de usuários em teste, fixture ou exemplo de documentação.

---

## 4. Testes por tipo

### 4.1 Unitários

- Máquina de estados de oportunidade, projeto, proposta e assinatura.
- Money/Decimal: soma, multiplicação, arredondamento, comparação.
- Elegibilidade: cada avaliador e precedência de outcome.
- Guardas de transição (ex.: publicar sem fonte).
- Derivação de `effectiveStatus` e `confirmationPending`.
- Hash de conteúdo e proveniência.
- Quota: reserva, confirmação, liberação.
- Normalização de e-mail, CNPJ, datas.

### 4.2 Integração

- API + Mongo real: índices únicos, transações de orçamento, soft delete.
- Redis real: rate limit, quota atômica, cache.
- Storage real (MinIO): upload, URL assinada, expiração.
- Filas reais: retry, DLQ, idempotência, outbox.
- Provider de IA fake: sucesso, JSON inválido, timeout, 429, content filter.
- Webhook: assinatura válida/inválida, duplicata, fora de ordem.

### 4.3 E2E (jornadas do PRD §13)

| Jornada | Cobertura mínima |
|:--|:--|
| J01 primeiro acesso | cadastro → verificação → perfil → dashboard |
| J02 encontrar edital | busca → filtro → detalhe → fonte → favorito |
| J03 analisar edital | upload → status → revisão → confirmação → elegibilidade |
| J04 criar projeto | criação → entrevista → seção → versão |
| J05 preparar submissão | validação → DOCX/PDF → marcar pronto → registrar submissão |

Fluxos negativos: oportunidade encerrada, quota excedida, documento infectado, versão congelada.

### 4.4 Segurança

| Teste | Critério |
|:--|:--|
| IDOR | tenant A não acessa recurso de B (404, não 403 com existência) |
| Escalada de papel | VIEWER não edita; ADMIN não vira OWNER |
| CSRF | mutação sem token é rejeitada |
| Rate limit | brute force bloqueado |
| Upload | executável renomeado, zip bomb, PDF com JS |
| Webhook | sem assinatura, replay, evento duplicado |
| Prompt injection | instrução em PDF não altera comportamento |
| Cross-tenant RAG | nenhum chunk de outro tenant |
| Segredos | logs e erros sem token/senha |

### 4.5 IA (avaliação)

Para cada tarefa (AI-001..AI-010):

- **Schema válido:** ≥ 99% após reparo.
- **Groundedness:** toda afirmação factual com evidência; taxa de campo sem evidência = 0 em `CONFIRMED`.
- **Missing information:** recall ≥ 95% em conjunto dourado.
- **Alucinação:** 0 nomes, valores ou datas inventados nos testes de regressão.
- **Custo:** dentro do orçamento por tarefa.
- **Injection:** 100% dos fixtures bloqueados.

Conjunto dourado versionado em `projeta-api/test/fixtures/ai/` (a criar na Fase 1).

### 4.6 Acessibilidade

- axe automatizado nas telas principais.
- Navegação por teclado em fluxos críticos.
- Leitor de tela: smoke manual por release.
- Contraste AA, foco visível, rótulos e mensagens de erro associadas.

### 4.7 Desempenho

- p95 < 500 ms em consultas simples; < 1 s em busca.
- Teste de carga em dashboard, lista de oportunidades e análise (fila).
- Nenhuma consulta sem índice em coleção relevante (verificação em CI com explain).

---

## 5. Dados de teste

- Factories: `makeUser`, `makeOrg`, `makeOpportunity`, `makeProject`, `makeVersion`, `makeSubscription`.
- Seed determinístico com `faker.seed`.
- Nenhum CPF/CNPJ real; usar geradores válidos fictícios.
- Fixtures de PDF: digital, escaneado, protegido, malicioso, gigante.
- Fixtures de IA: respostas válidas, inválidas, injection.

---

## 6. Pipeline de qualidade

```
PR:
  lint + typecheck
  unit
  integration (containers)
  contract (OpenAPI sem diff inesperado)
  security smoke (IDOR matrix, headers)
  build
  e2e smoke (jornada crítica)

main:
  tudo acima
  e2e completo
  acessibilidade (axe)
  análise de dependências

release:
  regressão completa
  carga (quando aplicável)
  evidências arquivadas
```

Falha em qualquer etapa bloqueia merge. Exceção exige registro e prazo.

---

## 7. Evidências

Cada entrega anexa:

1. requisito e spec;
2. lista de testes executados;
3. relatório de cobertura;
4. resultado de segurança;
5. link do run de CI;
6. riscos residuais.

Sem evidência, a entrega não é aceita.

---

## 8. Definition of Done (checklist operacional)

- [ ] Requisito e critério de aceite implementados.
- [ ] Testes unitários das regras novas.
- [ ] Teste de integração do fluxo.
- [ ] Teste de autorização (positivo e negativo).
- [ ] Casos de erro cobertos.
- [ ] Nenhuma regressão nos testes existentes.
- [ ] Contrato/OpenAPI atualizado.
- [ ] Logs e métricas no lugar.
- [ ] Documentação e changelog da spec atualizados.
- [ ] Evidência registrada.
- [ ] Revisão concluída.

---

## 9. Riscos de qualidade

| Risco | Mitigação |
|:--|:--|
| Testes frágeis de IA | provider fake + conjunto dourado + métricas, não snapshot de texto |
| Testes E2E lentos | poucos fluxos críticos, paralelismo, dados isolados |
| Vazamento de dado real | proibição explícita + revisão de fixture |
| Cobertura ilusória | foco em branches de domínio e autorização |
| Flaky | quarentena e correção antes de nova feature |
