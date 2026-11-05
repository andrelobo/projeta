# ADR-008 — Fontes oficiais como autoridade

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

O produto será usado para decidir participação em chamadas públicas. Informação errada gera prejuízo real ao usuário e risco reputacional ao PROJETA.

## Decisão

A **fonte oficial prevalece** sobre qualquer interpretação da IA. Toda oportunidade tem fonte, evidência e data de verificação. Dados não confirmados são marcados.

## Consequências

- Sem fonte verificável, não há publicação (RB-001).
- IA nunca corrige nem completa dado oficial sem evidência.
- O sistema distingue publicado, aberto e verificado (RB-019).
- Curadoria manual assistida é aceita no MVP.

## Alternativas consideradas

- Agregação livre de portais e blogs: rejeitada por confiabilidade.
- Confiar apenas em extração automática: rejeitada por risco de erro silencioso.
