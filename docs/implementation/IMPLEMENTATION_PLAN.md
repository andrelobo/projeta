# Plano de implementação — PROJETA

**Versão:** 1.0
**Status:** Aprovado em 2026-11-05 pelo Product Owner
**Regra:** nenhuma linha de aplicação é escrita antes da aprovação deste plano.

---

## 1. Premissas

1. O PRD 1.0 é a baseline de produto.
2. As specs estão em rascunho e serão aprovadas em blocos por milestone.
3. O MVP funcional é o domínio CULTURE.
4. Cada incremento é pequeno, testável, revisável e reversível.
5. Nenhum milestone começa sem os critérios de saída do anterior.
6. Decisões abertas bloqueantes devem ser resolvidas antes do milestone correspondente (§6).

---

## 2. Visão geral dos milestones

| # | Milestone | Specs | Resultado | Depende de |
|:--|:--|:--|:--|:--|
| M0 | Fundação técnica | SPEC-000 | dois repositórios no GitHub, ambiente, CI, shells, observabilidade | — |
| M1 | Identidade | SPEC-001 | cadastro, login, sessão, recuperação, exclusão | M0 |
| M2 | Organizações e tenancy | SPEC-002 | orgs, membros, RBAC, isolamento | M1 |
| M3 | Perfil e documentos | SPEC-003, SPEC-010 (núcleo) | perfil, upload/download seguro | M1, M2 |
| M4 | Oportunidades e Radar | SPEC-004, SPEC-013 (curadoria mínima) | catálogo, busca, favoritos, alertas | M2 |
| M5 | IA e analisador | SPEC-006, SPEC-005 | camada AI, análise de edital com evidências | M3, M4 |
| M6 | Elegibilidade | SPEC-007 | regras, avaliação, justificativas | M3, M5 |
| M7 | Projetos, propostas e versões | SPEC-008, SPEC-011 | projeto, geração, versão, submissão, dashboard | M5 |
| M8 | Exportação, billing e lançamento | SPEC-010 (export), SPEC-012, SPEC-013 | DOCX/PDF, assinatura, limites, admin completo | M7 |

> Ordem alternativa possível: M8 de billing antecipado para validar monetização. Não recomendada antes de M7 porque sem valor entregue não há conversão real.

---

## 3. Incrementos por milestone

### M0 — Fundação técnica

**Objetivo:** um esqueleto executável, observável e testável, sem regra de negócio.

| # | Incremento | Entrega verificável |
|:--|:--|:--|
| M0.1 | Bootstrap dos repositórios | criar `projeta-web` e `projeta-api` no GitHub, branch protection, TypeScript strict, eslint/prettier, `.nvmrc` |
| M0.2 | Config tipada | `projeta-api/src/config` com schema Zod e fail-fast; env do web na Vercel |
| M0.3 | API shell | NestJS, `/health/live`, `/health/ready`, Problem Details |
| M0.4 | Web shell | layout, tema, páginas de erro, acessibilidade base |
| M0.5 | Ambiente local | docker compose: Mongo, Redis, MinIO, Mailpit |
| M0.6 | Logs e correlação | pino JSON, request id, redação de PII |
| M0.7 | OpenAPI | geração a partir dos schemas, `/api/v1/docs` |
| M0.8 | Filas | BullMQ conectado, fila de exemplo, DLQ |
| M0.9 | Migrações | runner versionado + `schema_migrations` |
| M0.10 | CI | pipeline do api (lint, typecheck, test, build, OpenAPI) e do web (lint, typecheck, test, build) |
| M0.11 | Contratos publicados | `@projeta/contracts` no GitHub Packages + `openapi.json` versionado |
| M0.12 | Deploy staging | GitHub Actions: web via Vercel CLI; api/worker via GHCR + SSH como `lobojoe` na VPS; Caddy e compose de staging |
| M0.13 | Governança GitHub | CODEOWNERS, Environments `staging`/`production`, segredos, regras de merge, secret scanning/push protection, Dependabot e CodeQL (repos públicos) |

**Saída:** `projeta-web` e `projeta-api` sobem separadamente; staging acessível por domínio; contratos publicados; CI verde; health e docs funcionando; nenhum módulo de negócio.

---

### M1 — Identidade

| # | Incremento | Entrega |
|:--|:--|:--|
| M1.1 | Modelo de usuário e consentimentos | cadastro com termos/privacidade versionados |
| M1.2 | Senha e login | Argon2id, rate limit, erro genérico |
| M1.3 | Sessão e refresh rotativo | cookies, famílias, revogação |
| M1.4 | Verificação de e-mail | token, reenvio, Mailpit |
| M1.5 | Recuperação de senha | token único, TTL, sem enumeração |
| M1.6 | Conta | GET/PATCH `/users/me`, troca de senha, exclusão |
| M1.7 | Auditoria de identidade | eventos e consulta básica |

**Saída:** J01 completo; testes de segurança de auth verdes.

---

### M2 — Organizações e tenancy

| # | Incremento | Entrega |
|:--|:--|:--|
| M2.1 | Organização | CRUD, CNPJ, arquivamento |
| M2.2 | Membros | convite, aceite, remoção, papéis |
| M2.3 | Ownership | transferência, proteção do último owner |
| M2.4 | Tenant guard | `TenantContext`, repositório tipado |
| M2.5 | Matriz de capacidades | testes por papel e por recurso |
| M2.6 | Auditoria de tenancy | eventos de membership |

**Saída:** dois tenants isolados comprovadamente; matriz de acesso testada.

---

### M3 — Perfil e documentos

| # | Incremento | Entrega |
|:--|:--|:--|
| M3.1 | Perfil | CRUD, completude, histórico |
| M3.2 | Upload seguro | MIME real, hash, limite, storage privado |
| M3.3 | Varredura | scanStatus e bloqueio de download |
| M3.4 | Download assinado | URL curta, auditoria |
| M3.5 | Vínculo | perfil ↔ documento, organização ↔ documento |
| M3.6 | Categorias e listagem | filtros por categoria |

**Saída:** currículo e portfólio anexados e acessíveis só pelo dono.

---

### M4 — Oportunidades e Radar

| # | Incremento | Entrega |
|:--|:--|:--|
| M4.1 | Modelo de oportunidade | campos, `status`, `verificationStatus` |
| M4.2 | Fontes | CRUD, `allowed`, healthCheck |
| M4.3 | Evidências | registro e consulta por campo |
| M4.4 | Curadoria mínima | criar, revisar, publicar, verificar, arquivar |
| M4.5 | Temporal | job diário, `effectiveStatus`, `confirmationPending` |
| M4.6 | Busca e filtros | Atlas Search + filtros estruturados |
| M4.7 | Favoritos | idempotente, isolado |
| M4.8 | Buscas salvas | CRUD e casamento |
| M4.9 | Duplicidade | dedupeKey, fingerprint, fila de revisão |

**Saída:** radar com 20 oportunidades fictícias verificadas; nenhuma encerrada exibida como aberta.

---

### M5 — IA e analisador

| # | Incremento | Entrega |
|:--|:--|:--|
| M5.1 | Provider fake | interface e testes |
| M5.2 | Registry de prompts | versões, ativação, hash |
| M5.3 | Validação de saída | Zod, reparo, falha segura |
| M5.4 | Custo e quota | ledger, reserva, liberação |
| M5.5 | RAG | chunking, índice, filtro de tenant |
| M5.6 | Extração de PDF | texto, páginas, detecção de escaneado |
| M5.7 | OCR | integração com provedor (decisão aberta) |
| M5.8 | Análise estruturada | AI-001/AI-002, evidências, missing info |
| M5.9 | Revisão humana | editar, confirmar, versionar |
| M5.10 | Provider real | integração do provedor aprovado + fallback |

**Saída:** 5 editais reais analisados com precisão medida; prompt injection bloqueado.

---

### M6 — Elegibilidade

| # | Incremento | Entrega |
|:--|:--|:--|
| M6.1 | Modelo de regra | categorias, severidade, avaliadores |
| M6.2 | Regras draft | criação a partir da análise |
| M6.3 | Confirmação | revisão e versão |
| M6.4 | Engine | avaliadores determinísticos |
| M6.5 | Outcome | precedência e justificativas |
| M6.6 | UI | requisito × perfil × evidência |
| M6.7 | Reprodutibilidade | hash do conjunto e avaliação |

**Saída:** avaliação explicável e reproduzível; IA não altera outcome.

---

### M7 — Projetos, propostas e versões

| # | Incremento | Entrega |
|:--|:--|:--|
| M7.1 | Projeto | CRUD, draft, sectionPlan |
| M7.2 | Entrevista | AI-005, perguntas por lacuna |
| M7.3 | Seções | 19 seções, edição, status |
| M7.4 | Geração | AI-006 assíncrona, proveniência, missing info |
| M7.5 | Revisão | AI-007/008/009 |
| M7.6 | Versionamento | criar, listar, comparar, congelar |
| M7.7 | Propostas | vínculo múltiplo, snapshot |
| M7.8 | Submissão | registrar submissão e congelar |
| M7.9 | Dashboard | resumo, prazos, pendências |
| M7.10 | Notificações | in-app, preferências, e-mail |

**Saída:** J04 e J05 completas; versão congelada imutável comprovada.

---

### M8 — Exportação, billing e lançamento

| # | Incremento | Entrega |
|:--|:--|:--|
| M8.1 | Templates | DOCX/PDF versionados |
| M8.2 | Exportação | job, arquivo, download |
| M8.3 | Planos | catálogo e versões |
| M8.4 | Checkout | assinatura, trial |
| M8.5 | Webhooks | assinatura, idempotência, estado |
| M8.6 | Entitlements | feature, quota, reconciliação |
| M8.7 | Admin completo | usuários, planos, auditoria, DLQ |
| M8.8 | Hardening | revisão de segurança, acessibilidade, carga |
| M8.9 | Lançamento | checklist do PRD §20 |

**Saída:** critérios de lançamento do PRD §20 atendidos; billing em produção.

---

## 4. Dependências entre specs

```
SPEC-000
  ├── SPEC-001 ──► SPEC-002 ──► SPEC-003 ──► SPEC-010
  │                   │            │
  │                   └──► SPEC-004 ──► SPEC-005 ──► SPEC-007
  │                                      │
  │                                      └──► SPEC-008 ──► SPEC-009 (Fase 2)
  │                                                └──► SPEC-011
  ├── SPEC-006 ────────────────────────────────► (usada por 005 e 008)
  └── SPEC-012 ──► transversal ao MVP
      SPEC-013 ──► admin mínimo em M4; completo em M8
```

---

## 5. Critérios de saída por milestone

Um milestone só é concluído quando:

1. Todos os incrementos planejados estão implementados.
2. Testes unitários, integração e E2E do escopo passam.
3. Testes de autorização e IDOR do escopo passam.
4. OpenAPI e documentação atualizados.
5. Nenhum bug crítico aberto.
6. Demo executável em staging.
7. Evidências arquivadas em `docs/quality/`.
8. Aprovação humana registrada.

---

## 6. Decisões bloqueantes por milestone

| Milestone | Decisão necessária | ID |
|:--|:--|:--|
| M0 | ferramenta de migração; observabilidade; criação dos repositórios em `andrelobo`; usuário `lobojoe` na VPS | OPEN-002, OPEN-008 |
| M1 | validade de tokens; MFA para admin; auth em previews | OPEN-011, OPEN-010, OPEN-051 |
| M3 | storage definitivo e região | OPEN-003 |
| M3 | retenção de documentos | OPEN-023 |
| M4 | vocabulário de modalidade; janela de verificação | OPEN-016, OPEN-018 |
| M5 | provedor de IA; OCR; embeddings | OPEN-048, OPEN-021, OPEN-024 |
| M5 | limite de PDF e confiança | OPEN-020, OPEN-022 |
| M6 | quem confirma regras | OPEN-027 |
| M7 | autosave vs versão explícita | OPEN-030 |
| M8 | provedor de pagamento; limites e preços | OPEN-001, OPEN-025 |
| M8 | política de trial e cancelamento | OPEN-042 |
| Transversal | licença, retenção de logs/auditoria | OPEN-047, OPEN-009, OPEN-045 |

---

## 7. Riscos do plano

| Risco | Prob. | Impacto | Mitigação |
|:--|:--|:--|:--|
| Escopo do MVP crescer | alta | alto | spec aprovada por milestone; mudança exige ADR |
| Qualidade da extração de edital | alta | alto | conjunto dourado, revisão humana obrigatória |
| Custo de IA acima do previsto | média | alto | orçamento, cache, modelos menores por tarefa |
| Integração de pagamento | média | alto | provider sandbox desde M8.1; webhooks testados |
| Vazamento de dado sensível | baixa | crítico | storage privado, testes de IDOR, revisão LGPD |
| Exposição em repositórios públicos | média | alto | secret scanning, revisão do que entra em `docs/`, licença definida |
| Atraso por decisões abertas | alta | médio | resolver bloqueios antes do milestone |
| Complexidade de multi-tenancy | média | médio | fundação cedo (M2) e testes desde o início |

---

## 8. O que este plano não faz

- Não define datas ou prazos comerciais.
- Não define fornecedores.
- Não autoriza implementação.
- Não substitui as specs.
- Não cobre a Fase 2 (orçamento e cronograma — SPEC-009), nem HEALTH, marketplace, colaboração avançada ou analytics (Fases 3–4).

---

## 9. Aprovação

Para iniciar M0, são necessários:

- [x] Aprovação deste plano (2026-11-05).
- [x] ADR-012 a ADR-025 aprovados (ADR-011 rejeitado).
- [ ] Aprovação das specs SPEC-000 a SPEC-004 (primeiro bloco).
- [ ] Resolução de OPEN-002 e OPEN-008.
- [x] Criação dos três repositórios em `github.com/andrelobo`.
- [x] Licença definida (Apache-2.0).
- [ ] Confirmação de que não há dado real de cliente no ambiente de desenvolvimento.
