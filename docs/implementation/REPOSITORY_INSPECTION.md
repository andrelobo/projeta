# Relatório de inspeção do repositório

**Data:** 2026-11-05
**Papel:** System Architect (Fase 1 do *Initial Engineering Task*)
**Objetivo:** inspecionar o workspace antes de qualquer decisão de arquitetura ou implementação.

---

## 1. O que existia

O workspace da sessão (`<workspace>`) continha, antes desta tarefa:

| Item | Natureza |
|:--|:--|
| `ANALISE_<acervo>_2026-11-05.md` | análise de um acervo documental externo |
| `INVENTARIO_ARQUIVOS_2026-11-05.csv` | inventário de 85 arquivos do acervo externo |
| `_extract/` | textos extraídos dos PDFs/DOCX/XLSX daquele acervo |

Nada relacionado ao PROJETA. Não existia `projeta/`, `package.json`, código-fonte, banco, container ou configuração.

O acervo pai (`../`) é o dossiê de um projeto cultural real, usado como contexto de problema — **não** como base de dados do produto e **não** incluído neste repositório.

## 2. Ambiente

| Item | Valor |
|:--|:--|
| Sistema | Linux Mint 22.3 (kernel 7.0.0-42) |
| Node.js | v22.23.1 |
| pnpm | 11.15.1 |
| npm | 12.0.1 |
| Docker | 29.9.3 |
| Git | 2.43.0 |
| Disco livre | ~15 GB |
| Locale | pt_BR.UTF-8 |
| Workspace | `<workspace>` |
| Controle de versão | **nenhum** (`git rev-parse` falhou) |

## 3. Achados

1. **Não é um repositório git.** Não há histórico, branch, `.gitignore` nem política de revisão. Isso precisa ser resolvido antes de M0.
2. **Não há código de produto.** Nenhum artefato de PROJETA a preservar, migrar ou respeitar.
3. **Há documentação vizinha de outro projeto** (acervo externo). Fica fora de `projeta/` e não deve ser misturada, publicada ou usada como seed.
4. **Toolchain compatível** com a stack proposta (Node 22, pnpm, Docker).
5. **Disco suficiente** para imagens Docker e build local, com folga moderada (15 GB).
6. **Sem CI, sem secrets, sem banco.** Nada a migrar.

## 4. Decisão de localização

Criar o **repositório de documentação** (owner `andrelobo`) em:

```
<workspace>/projeta/
```

Os repositórios de código serão separados no GitHub (ADR-021): `projeta-web` (Vercel) e `projeta-api` (VPS).

**Motivos:**

- é o diretório de trabalho da sessão e o único com escrita garantida pela política do ambiente;
- mantém a documentação do PROJETA isolada do acervo externo;
- permite mover a pasta para um repositório próprio sem alterar caminhos internos.

**Consequência:** os caminhos nos documentos são relativos à raiz `projeta/`. Se a pasta for movida, nenhum link interno quebra.

## 5. O que foi criado

```
projeta/
├── AGENTS.md
├── CONTEXT.md
├── README.md
└── docs/
    ├── product/
    │   ├── PRD.md
    │   ├── PRD_ERRATA.md
    │   └── GLOSSARY.md
    ├── architecture/
    │   ├── SYSTEM_DESIGN.md
    │   ├── DOMAIN_MODEL.md
    │   └── ADR/ (24 ADRs + índice)
    ├── specifications/ (14 specs + template + índice)
    ├── implementation/
    │   ├── REPOSITORY_INSPECTION.md
    │   └── IMPLEMENTATION_PLAN.md
    ├── quality/
    │   └── TEST_STRATEGY.md
    └── validation/
        ├── CONSISTENCY_REPORT.md
        └── OPEN_DECISIONS.md
```

Os repositórios de aplicação (`projeta-web` e `projeta-api`) serão criados no GitHub em M0, separados deste.

Nenhum arquivo pré-existente foi alterado, movido ou removido.

## 6. O que **não** foi feito (por decisão)

- Nenhuma linha de código de aplicação.
- Nenhum `package.json`, lockfile ou instalação de dependência.
- Nenhum `git init`, commit ou branch.
- Nenhum container iniciado.
- Nenhum schema de banco criado.
- Nenhum provedor escolhido (IA, pagamento, storage, OCR, e-mail).
- Nenhuma decisão de preço ou limite de plano.

## 7. Riscos e pendências

| Risco | Impacto | Encaminhamento |
|:--|:--|:--|
| Ausência de git | perda de histórico, revisão frágil | criar os três repositórios no GitHub antes de M0 |
| Sem CI | regressão silenciosa | pipelines separados em M0.10 |
| Usuário da VPS (`lobojoe`) ainda não provisionado | deploy via SSH falha | criar usuário, grupo docker e diretório `/home/lobojoe/projeta` |
| Repositórios públicos | expõem código, specs e roadmap | licença definida, secret scanning, revisão editorial do `docs/` |
| Docker local pode exigir imagens grandes | setup lento | pré-baixar imagens na Fase 0 |
| 15 GB livres | limite para builds e imagens | monitorar; limpar cache |
| Documentação vizinha (acervo externo) | confusão de escopo | manter fora do repo e do seed |
| Decisões abertas bloqueantes | atraso em M1/M3/M5/M8 | resolver antes do milestone (§6 do plano) |

## 8. Próximo passo

Aprovação humana do plano e das specs do primeiro bloco (SPEC-000 a SPEC-004) para iniciar M0.
