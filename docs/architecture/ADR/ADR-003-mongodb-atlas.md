# ADR-003 — MongoDB Atlas como banco inicial

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

O domínio é documental: editais com estrutura variável, versões de proposta, análises com evidências por campo e metadados heterogêneos. O time já tem familiaridade com MongoDB.

## Decisão

Usar **MongoDB Atlas** como banco principal, com `Decimal128` para dinheiro, ObjectId como identificador e Atlas Search para busca textual.

## Consequências

- Schema validado na aplicação (Zod + Mongoose), não no banco.
- Transações multi-documento disponíveis (replica set), usadas em orçamento e billing.
- Índices precisam ser projetados com cuidado; crescimento de coleções exige monitoramento.
- Relatórios analíticos complexos podem exigir agregação dedicada no futuro.

## Alternativas consideradas

- PostgreSQL: forte em integridade relacional, mas exigiria modelagem mais rígida para conteúdo versionado e análise de edital.
- Banco híbrido: complexidade não justificada no MVP.
