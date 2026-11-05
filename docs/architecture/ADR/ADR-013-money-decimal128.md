# ADR-013 — Dinheiro em Decimal128 + tipo Money

- **Status:** Proposto — requer aprovação
- **Data:** 2026-11-05
- **Decisores:** System Architect
- **Relacionado:** RB-004, RF-072

## Contexto

Orçamentos, limites de edital e cobrança exigem exatidão. `float`/`number` produz erro de arredondamento e é proibido pelo PRD.

## Decisão

Persistir valores monetários como **Decimal128** (BSON) e manipulá-los com o tipo `Money` (moeda + valor decimal) em `src/shared` do `projeta-api`. Cálculos usam biblioteca decimal, nunca `Number` para soma.

## Justificativa

- Evita erro de centavos em orçamento e cobrança.
- Decimal128 é nativo do MongoDB e ordenável.
- `Money` encapsula moeda e impede soma de moedas diferentes.

## Consequências

- Conversão explícita nas bordas (API ↔ JSON string, não number).
- Serialização de API expõe valor como string decimal + moeda.
- Testes de arredondamento obrigatórios.
- Relatórios podem precisar de agregação com `$toDecimal`.

## Alternativas

- Centavos inteiros: também exato, mas pior para valores fracionários e limites de edital.
- `number`: rejeitado por precisão.
