# ADR-020 — Registry versionado de prompts e livro-razão de custo de IA

- **Status:** Aceito (2026-11-05, aprovado com o plano)
- **Data:** 2026-11-05
- **Decisores:** System Architect
- **Relacionado:** ADR-005, RB-010, §7.6, RNF-021

## Contexto

Resultados de IA precisam ser reproduzíveis e auditáveis; custo precisa ser controlado por tenant. Prompt solto no código impede ambos.

## Decisão

- `PromptTemplate` versionado por `taskId`, com status `DRAFT|ACTIVE|RETIRED`.
- Toda chamada grava `ai_interactions` com prompt/versão, modelo, tokens, latência, custo e hash das entradas.
- Orçamento por operação e por tenant/dia; excedeu → erro tratado (não truncar silenciosamente).

## Justificativa

- Permite comparar qualidade entre versões e reverter.
- Torna o custo rastreável e atribuível.
- Sustenta auditoria e resposta a incidente.

## Consequências

- Mudança de prompt exige nova versão e aprovação.
- Cache de resposta por `hash(entrada) + promptVersion` reduz custo.
- Métricas de custo entram no painel administrativo.

## Alternativas

- Prompts no código: simples, mas sem histórico nem experimento controlado.
- Sem registro de custo: inviável para o modelo de negócio.
