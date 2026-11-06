# CONTEXT.md — PROJETA

> Memória curta do projeto. Atualizar ao final de cada milestone ou decisão relevante.
> Não substitui PRD, ADRs ou specs — apenas orienta quem chega agora.

- Atualizado em: 2026-11-06
- Milestone: **Engineering Foundation 0.1**
- Estado: **plano, specs e decisões de fundação aprovados; M0 desbloqueado**
- Próxima ação: executar M0.4 (web shell com tema, páginas de erro e acessibilidade)

---

## 1. O que é o PROJETA

Plataforma SaaS brasileira que ajuda artistas, produtores culturais, consultores, coletivos e organizações a:

1. encontrar oportunidades de financiamento;
2. interpretar editais com evidência e rastreabilidade;
3. verificar elegibilidade;
4. desenvolver propostas;
5. organizar documentação;
6. exportar submissões;
7. administrar projetos aprovados.

Dois domínios independentes: **CULTURE** (MVP) e **HEALTH** (futuro, não funcional no MVP).

## 2. Onde estamos

| Item | Estado |
|:--|:--|
| PRD 1.0 | Transcrito em `docs/product/PRD.md` |
| Modelo de domínio | `docs/architecture/DOMAIN_MODEL.md` |
| Arquitetura de sistema | `docs/architecture/SYSTEM_DESIGN.md` |
| ADRs | 25 (ADR-001 a ADR-010 do PRD; ADR-011 rejeitado; ADR-012 a ADR-025 aceitos) |
| Especificações | 14 + template; SPEC-000 a SPEC-004 aprovadas; SPEC-005 a SPEC-013 em rascunho |
| Plano incremental | `docs/implementation/IMPLEMENTATION_PLAN.md` |
| Estratégia de testes | `docs/quality/TEST_STRATEGY.md` |
| Consistência e lacunas | `docs/validation/CONSISTENCY_REPORT.md` |
| Reconciliação de status (2026-11-06) | `docs/validation/STATUS_RECONCILIATION_2026-11-06.md` |
| Decisões abertas | `docs/validation/OPEN_DECISIONS.md` |
| Progresso do M0 | `docs/implementation/M0_PROGRESS.md` |
| Código | Fundação M0.1 a M0.3 em `projeta-api` e `projeta-web`; sem funcionalidade de produto |

## 3. Decisões já tomadas

### 3.1 Pelo PRD (não reabrir sem ADR)

- Monólito modular NestJS (ADR-001).
- Next.js no frontend (ADR-002).
- MongoDB Atlas (ADR-003).
- Redis + BullMQ (ADR-004).
- LLM desacoplado de provedor (ADR-005).
- CULTURE primeiro (ADR-006).
- HEALTH isolado e futuro (ADR-007).
- Fontes oficiais como autoridade (ADR-008).
- Multi-tenancy desde a fundação (ADR-009).
- SDD obrigatório (ADR-010).

### 3.2 Decisões aprovadas nesta fase

- **Polyrepo no GitHub**: `projeta` (docs), `projeta-web` (Vercel) e `projeta-api` (VPS), sob o owner `andrelobo` (conta pessoal), repositórios **públicos**. Monorepo rejeitado pelo Product Owner (ADR-021).
- **Licença Apache-2.0** nos três repositórios (ADR-025).
- **Todo deploy é via GitHub Actions** (ADR-024): Vercel por CLI e API por GHCR + SSH na VPS. Imagem GHCR privada.
- `projeta-api` publica `@projeta/contracts` no GitHub Packages; `projeta-web` fixa a versão exata e usa `NODE_AUTH_TOKEN` no build.
- Deploy: web na Vercel; API/worker em VPS com Docker Compose e Caddy (ADR-022). A VPS é operada pelo usuário `lobojoe`.
- PRD recebeu a Emenda 01 (`docs/product/PRD_ERRATA.md`), que substitui o §9.1.

### 3.3 Aprovadas junto com o plano (2026-11-05)

- Autenticação cross-domain e previews (ADR-023).
- Contratos primeiro: Zod como fonte de verdade, OpenAPI gerado.
- Dinheiro em `Decimal128` no banco + `Money` em minor units na aplicação.
- Autenticação por cookie `httpOnly` + rotação de refresh + CSRF.
- Propriedade de recurso como `ownerType/ownerId` (USER ou ORGANIZATION).
- Status efetivo de oportunidade derivado da data, com `verificationStatus` separado.
- Proposta = `project × opportunity`; versões pertencem à proposta, não ao projeto.
- Jobs idempotentes com chave de idempotência e DLQ.
- Entitlements aplicados no servidor com contadores atômicos.
- Registry versionado de prompts e livro-razão de custo de IA.

### 3.4 Aprovadas em 2026-11-06

- Migração por script versionado simples com `schema_migrations` (OPEN-002).
- Observabilidade com OpenTelemetry e backend gerenciado via OTLP (OPEN-008).
- Emenda 02 ratificada; ADR-023 vigente (OPEN-053).
- `SYSTEM_DESIGN`, `DOMAIN_MODEL` e `TEST_STRATEGY` ratificados (OPEN-054).
- Token `NODE_AUTH_TOKEN` para leitura de `@projeta/contracts` na Vercel.
- Correção editorial do ADR-021: repositórios públicos.
- Confirmado que os três repositórios não contêm dados reais de cliente.

## 4. O que NÃO fazer agora

- Não criar código fora do escopo do milestone aprovado.
- Não reintroduzir monorepo.
- Não adicionar dependência de produção sem registro e aprovação.
- Não definir fornecedor de IA, pagamento, e-mail, OCR ou storage definitivo.
- Não implementar o domínio HEALTH.
- Não prometer submissão automática em portais.
- Não usar dados do acervo externo (pasta acima) como seed sem autorização — são dados reais de terceiros.

## 5. Restrições de contexto que afetam decisões

1. O produto nasceu da dor real documentada em um acervo externo de projeto cultural: orçamento fechado "de cima para baixo", prazos perdidos, editais mal interpretados, documentação dispersa. O PROJETA deve resolver exatamente esses pontos — e o PRD já os reflete.
2. O mercado inicial é Brasil, pt-BR, com forte presença de MEIs, coletivos e organizações pequenas.
3. O produto não pode prometer aprovação nem substituir a decisão do usuário.
4. O produto deve ser comercialmente sustentável; custo de IA é variável crítica.

## 6. Glossário rápido

- **Oportunidade**: chamada pública ou privada de financiamento, com fonte verificável.
- **Edital**: documento oficial que define regras da oportunidade.
- **Análise de edital**: extração estruturada com evidências.
- **Elegibilidade**: avaliação determinística de compatibilidade perfil × requisitos.
- **Projeto**: conteúdo autoral do usuário, independente de edital.
- **Proposta**: instância do projeto vinculada a uma oportunidade, versionada e congelável.
- **Versão congelada**: snapshot imutável de proposta submetida.
- **Entitlement**: direito de uso derivado do plano (limite ou feature).
- **Tenant**: usuário ou organização que possui recursos privados.

Glossário completo: `docs/product/GLOSSARY.md`.

## 7. Próximo passo

O plano, o primeiro bloco de specs e as decisões de fundação estão aprovados em `docs/implementation/APPROVALS.md`. O **M0.1** (bootstrap), o **M0.2** (config tipada) e o **M0.3** (API shell) foram concluídos em 2026-11-06 — evidências em `docs/implementation/M0_PROGRESS.md`.

Continuar o M0 conforme `docs/implementation/IMPLEMENTATION_PLAN.md`:

1. ~~bootstrap dos repositórios `projeta-web` e `projeta-api`~~;
2. ~~config tipada~~ (ambiente local é M0.5);
3. ~~health e Problem Details~~; faltam logs (M0.6) e OpenAPI (M0.7);
4. web shell (M0.4), ambiente local, CI e harness de testes;
5. só então identidade e tenancy.
