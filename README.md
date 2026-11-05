# PROJETA

**Project Intelligence & Management Platform** — plataforma SaaS para encontrar oportunidades de financiamento, interpretar editais, verificar elegibilidade, desenvolver propostas e administrar projetos financiados.

- Mercado inicial: Brasil · pt-BR
- Domínio do MVP: **CULTURE**
- Domínio futuro: **HEALTH** (previsto na arquitetura, não implementado)
- Modelo: Freemium + assinatura
- Stack: Next.js · TypeScript · NestJS · MongoDB · Redis · LLM APIs

> **Estado do repositório: documentação apenas.**
> Nenhuma linha de aplicação foi escrita. A implementação depende de aprovação do plano em [`docs/implementation/IMPLEMENTATION_PLAN.md`](docs/implementation/IMPLEMENTATION_PLAN.md).

---

## Comece por aqui

| Se você é… | Leia |
|:--|:--|
| Produto / negócio | [`docs/product/PRD.md`](docs/product/PRD.md), [`docs/product/GLOSSARY.md`](docs/product/GLOSSARY.md) |
| Arquitetura | [`docs/architecture/SYSTEM_DESIGN.md`](docs/architecture/SYSTEM_DESIGN.md), [`docs/architecture/DOMAIN_MODEL.md`](docs/architecture/DOMAIN_MODEL.md) |
| Implementação | [`docs/specifications/README.md`](docs/specifications/README.md), [`docs/implementation/IMPLEMENTATION_PLAN.md`](docs/implementation/IMPLEMENTATION_PLAN.md) |
| Qualidade | [`docs/quality/TEST_STRATEGY.md`](docs/quality/TEST_STRATEGY.md) |
| Agente de IA | [`AGENTS.md`](AGENTS.md), [`CONTEXT.md`](CONTEXT.md) |
| Decisões e lacunas | [`docs/validation/CONSISTENCY_REPORT.md`](docs/validation/CONSISTENCY_REPORT.md), [`docs/validation/OPEN_DECISIONS.md`](docs/validation/OPEN_DECISIONS.md) |

## Repositórios

| Repositório | Conteúdo | Deploy |
|:--|:--|:--|
| [`projeta`](https://github.com/andrelobo/projeta) | documentação (este repositório) | — |
| [`projeta-web`](https://github.com/andrelobo/projeta-web) | Next.js + TypeScript | GitHub Actions → Vercel |
| [`projeta-api`](https://github.com/andrelobo/projeta-api) | NestJS + worker + infraestrutura | GitHub Actions → VPS (Docker + Caddy) |

Os três repositórios são **públicos** em `github.com/andrelobo`; a imagem do GHCR permanece privada. O deploy na VPS usa o usuário `lobojoe`.

Contratos: `projeta-api` publica `@projeta/contracts` (GitHub Packages) e `openapi.json`; `projeta-web` consome versão exata.

## Estrutura

```
projeta/
├── AGENTS.md          # regras de trabalho para agentes
├── CONTEXT.md         # estado atual e memória curta
├── README.md
└── docs/
    ├── product/       # PRD, glossário
    ├── architecture/  # system design, domínio, ADRs
    ├── specifications/# specs implementáveis (SPEC-000..SPEC-013)
    ├── implementation/# inspeção, plano incremental
    ├── quality/       # estratégia de testes
    └── validation/    # consistência, decisões abertas
```

**Não há monorepo.** O código vive em dois repositórios separados no GitHub — `projeta-web` (Vercel) e `projeta-api` (VPS) — criados somente na FASE 0, após aprovação. Ver ADR-021 e ADR-022.

## Escopo do MVP (Fase 1)

- Cadastro, login e perfil profissional
- Organizações, papéis e isolamento multi-tenant
- Radar de oportunidades com fonte verificável
- Analisador de editais em PDF com evidências
- Motor de elegibilidade explicável
- Assistente de projeto com IA e versionamento
- Exportação DOCX/PDF
- Assinatura e limites de plano
- Dashboard operacional

Fora do MVP: marketplace, rede social, submissão automática, prestação de contas avançada, HEALTH funcional, app nativo.

## Roadmap resumido

| Fase | Entrega |
|:--|:--|
| 0 | Fundação: repositórios, CI, configuração, identidade, banco |
| 1 | MVP Cultural |
| 2 | Produtividade: orçamento, cronograma, documentos, alertas |
| 3 | Domínio Saúde |
| 4 | Expansão: marketplace, colaboração, integrações |

Detalhe em [`docs/product/PRD.md`](docs/product/PRD.md) §17.

## Como contribuir

1. Leia `AGENTS.md`.
2. Verifique se existe spec aprovada para a mudança.
3. Mudou arquitetura? Abra ADR.
4. Mudou requisito? Atualize PRD/spec e registre no changelog.
5. Teste antes de declarar concluído.

## Licença

[Apache License 2.0](LICENSE) — © 2026 André Lobo (andrelobo).

A licença cobre código e documentação; marca e identidade visual não são licenciadas (ADR-025).
