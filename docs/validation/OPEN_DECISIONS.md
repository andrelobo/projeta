# Decisões abertas — PROJETA

**Data:** 2026-11-05
**Regra:** nenhuma decisão desta lista pode ser resolvida por suposição em código. Cada uma exige registro (ADR, ata ou decisão de produto) e atualização deste arquivo.

**Como fechar:** mover a linha para “Decisões encerradas”, registrar data, responsável, decisão e documento que a formalizou.

---

## 1. Bloqueantes por milestone

| Milestone | Decisões |
|:--|:--|
| M0 | — (OPEN-002 e OPEN-008 resolvidas em 2026-11-06) |
| M1 | OPEN-010, OPEN-011 |
| M3 | OPEN-003, OPEN-023 |
| M4 | OPEN-016, OPEN-018 |
| M5 | OPEN-021, OPEN-024, OPEN-048, OPEN-020, OPEN-022 |
| M6 | OPEN-027, OPEN-028 |
| M7 | OPEN-030 |
| M8 | OPEN-001, OPEN-025, OPEN-042 |
| Transversal | OPEN-009, OPEN-045, OPEN-050 |

---

## 2. Lista canônica

| ID | Tema | Pergunta | Impacto | Opções | Recomendação | Milestone |
|:--|:--|:--|:--|:--|:--|:--|
| OPEN-001 | Pagamento | Qual provedor de assinatura? | alto | Stripe, Iugu, Asaas, Pagar.me | decidir por suporte a Pix + cartão + webhooks confiáveis | M8 |
| OPEN-003 | Storage | Provedor e região definitivos? | alto | S3, R2, Backblaze, GCS | região com latência baixa no Brasil e custo previsível | M3 |
| OPEN-004 | Cronograma | `stage` é entidade ou string? | médio | entidade `project_stages` vs string versionada | começar como string versionada; entidade só se houver necessidade | M7 |
| OPEN-005 | Projeto | Haverá coautoria/múltiplos donos? | médio | dono único vs coautores | manter dono único no MVP; coautoria na Fase 2 | M7 |
| OPEN-006 | Projeto | Reabrir projeto arquivado? | baixo | reabrir vs criar novo | criar novo projeto; arquivado é terminal | M7 |
| OPEN-007 | Tenancy | Transferir projeto de usuário para organização? | médio | sim, auditado vs não | permitir com confirmação dupla e auditoria | M7 |
| OPEN-009 | LGPD | Retenção de logs? | médio | 30/90/180 dias | 90 dias, com exceção para segurança | M0 |
| OPEN-010 | Segurança | MFA obrigatório para ADMIN? | médio | opcional vs obrigatório | obrigatório para PLATFORM_ADMIN; opcional para organização | M1 |
| OPEN-011 | Segurança | Validade de access/refresh? | baixo | 15 min/30 d vs outros | 15 min/30 d, calibrar em staging | M1 |
| OPEN-012 | Planos | Limite de membros por plano? | médio | 1/3/10/ilimitado | definir junto com OPEN-025 | M8 |
| OPEN-013 | Organização | Múltiplos CNPJ por organização? | baixo | sim vs não | não no MVP | M2 |
| OPEN-014 | Perfil | Vocabulário de áreas: controlado ou livre? | médio | taxonomia vs tags livres | taxonomia com fallback livre | M3 |
| OPEN-015 | Perfil | Múltiplos perfis por usuário (PF + PJ)? | alto | um vs vários | um perfil por usuário no MVP; PJ via organização | M3 |
| OPEN-016 | Radar | Vocabulário de modalidade? | médio | edital, prêmio, bolsa, chamada, credenciamento | lista acima, extensível | M4 |
| OPEN-017 | Radar | Oportunidades visíveis sem login? | alto | público vs autenticado | autenticado no MVP; SEO na Fase 4 | M4 |
| OPEN-018 | Radar | Janela de verificação válida? | médio | 7/15/30/60 dias | 30 dias, ajustável por fonte | M4 |
| OPEN-019 | Fontes | Quais fontes automatizar primeiro? | alto | manual vs conector | validar termos de uso antes de qualquer conector | Fase 2 |
| OPEN-020 | PDF | Limite de tamanho e páginas? | médio | 10/20/50 MB; 100/300/500 pág. | 20 MB e 300 páginas | M5 |
| OPEN-021 | OCR | Serviço de OCR? | alto | Tesseract local, Google Vision, Textract | avaliar custo/qualidade com 10 editais escaneados | M5 |
| OPEN-022 | IA | Limiar de confiança para revisão? | médio | 0,5/0,6/0,7 | 0,6 com revisão obrigatória abaixo | M5 |
| OPEN-023 | LGPD | Retenção de documentos e PDFs? | alto | 1/2/5 anos | definir com jurídico; proposta submetida por 5 anos | M3 |
| OPEN-024 | IA | Embeddings e vector store? | alto | Atlas Vector Search, pgvector, Pinecone | Atlas Vector Search para evitar novo componente | M5 |
| OPEN-025 | Comercial | Preços e limites dos planos? | alto | — | definir após entrevistas com P01/P02 | M8 |
| OPEN-026 | LGPD | Retenção de prompts e entradas? | médio | 30/90/365 dias | 90 dias, com hash para auditoria | M5 |
| OPEN-027 | Elegibilidade | Quem confirma regras? | alto | curador vs usuário | curador (PLATFORM_OPERATOR) no MVP | M6 |
| OPEN-028 | Elegibilidade | Oportunidade sem regras estruturadas? | alto | bloquear vs TEXT_MANUAL | permitir avaliação manual com aviso | M6 |
| OPEN-029 | Elegibilidade | CNAE/atividade é requisito avaliável? | médio | sim vs não | não no MVP; dados insuficientes | M6 |
| OPEN-030 | Projetos | Autosave ou versão explícita? | médio | autosave + checkpoint vs só explícito | autosave no draft + versão explícita | M7 |
| OPEN-031 | Projetos | Limite de caracteres por seção? | baixo | por seção vs por edital | extrair do edital; sem limite global | M7 |
| OPEN-032 | Projetos | Modelos de seção por oportunidade? | médio | genérico vs por edital | genérico no MVP | Fase 2 |
| OPEN-034 | Orçamento | Arredondamento? | baixo | half-up vs banker's | half-up, documentado | M8 |
| OPEN-035 | Orçamento | Limites por categoria vêm do edital? | médio | sim vs não | sim, quando extraídos | Fase 2 |
| OPEN-036 | Cronograma | Etapas mapeadas do edital? | médio | fixas vs por edital | por edital, com fallback | Fase 2 |
| OPEN-037 | Exportação | Como renderizar PDF? | médio | LibreOffice, Gotenberg, serviço | Gotenberg em container | M8 |
| OPEN-038 | Planos | Limite de storage por plano? | médio | 100 MB/1 GB/10 GB | definir com OPEN-025 | M8 |
| OPEN-039 | E-mail | Provedor transacional? | médio | Resend, SES, Postmark | decidir por entregabilidade e custo | M1 |
| OPEN-040 | Notificações | Frequência de e-mail? | baixo | imediato vs digest | digest diário para oportunidades | M7 |
| OPEN-041 | Notificações | Alertas por organização? | médio | usuário vs org | usuário no MVP | M7 |
| OPEN-042 | Comercial | Trial e cancelamento? | médio | 7/14/30 dias; imediato vs fim do período | 14 dias; cancelar no fim do período | M8 |
| OPEN-043 | Comercial | Cobrança por assento no STUDIO? | médio | por assento vs flat | flat no MVP | M8 |
| OPEN-044 | Admin | Impersonação de usuário? | alto | sim vs não | não no MVP | Fase 4 |
| OPEN-045 | LGPD | Retenção de auditoria? | médio | 1/2/5 anos | 2 anos para admin/financeiro | Transversal |
| OPEN-046 | Admin | Restringir admin por IP/VPN? | médio | sim vs não | sim para PLATFORM_ADMIN em produção | M8 |
| OPEN-048 | IA | Provedor principal e fallback? | alto | OpenAI, Anthropic, DeepSeek, Google | avaliar qualidade/custo em 10 editais antes de decidir | M5 |
| OPEN-049 | Escopo | M08/M09 são MVP ou Fase 2? | alto | MVP vs Fase 2 | Fase 2, conforme roadmap §17 | M0 |
| OPEN-050 | Operação | RPO/RTO de backup? | médio | 24h/4h; 8h/1h | RPO 24h, RTO 8h no MVP | M0 |

> Observação: `OPEN-033` não é utilizado; a numeração canônica segue de `OPEN-032` para `OPEN-034`.

---

## 3. Decisões já encerradas

| ID | Data | Decisão | Autoridade | Documento |
|:--|:--|:--|:--|:--|
| OPEN-052 | 2026-11-05 | Repositórios no GitHub sob o owner **`andrelobo`** (conta pessoal), **públicos**; deploy via GitHub Actions; VPS operada por `lobojoe` | Product Owner | ADR-021, ADR-022, ADR-024 |
| OPEN-047 | 2026-11-05 | Licença **Apache-2.0** nos três repositórios | Product Owner (delegou) | ADR-025 |
| OPEN-051 | 2026-11-05 | Previews da Vercel em modo mock; BFF apenas se necessário | Product Owner (com o plano) | ADR-023 |
| OPEN-002 | 2026-11-06 | Runner de migração: script versionado simples com coleção `schema_migrations` | Product Owner | SPEC-000 §4.9 |
| OPEN-008 | 2026-11-06 | Observabilidade: OpenTelemetry com backend gerenciado via OTLP; fornecedor definido até M0.6 | Product Owner | SPEC-000 §7, SYSTEM_DESIGN §13 |
| OPEN-053 | 2026-11-06 | Emenda 02 ratificada e ADR-023 vigente | Product Owner | `PRD_ERRATA.md`, ADR-023 |
| OPEN-054 | 2026-11-06 | `SYSTEM_DESIGN`, `DOMAIN_MODEL` e `TEST_STRATEGY` ratificados | Product Owner | respectivos documentos |

As decisões arquiteturais estão em ADR-001 a ADR-025; ADR-011 foi rejeitado e as demais estão aceitas.

---

## 4. Critério para considerar uma decisão encerrada

1. Responsável nomeado.
2. Opção escolhida e justificada.
3. Impacto em prazo/custo/arquitetura avaliado.
4. Documento atualizado (ADR, spec ou este arquivo).
5. Comunicação registrada ao time.

Sem os cinco itens, a decisão continua aberta.
