# ADR-007 — HEALTH como domínio futuro isolado

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

Projetos de saúde podem envolver dados pessoais sensíveis, regras institucionais próprias e requisitos regulatórios distintos.

## Decisão

Tratar HEALTH como domínio **isolado**, com regras, fontes, modelos e políticas de dados próprias. Não reutilizar regras de elegibilidade cultural por conveniência.

## Consequências

- A camada de elegibilidade precisa ser genérica o suficiente para receber regras por domínio.
- Dados de saúde exigirão análise de privacidade (DPIA) antes do lançamento.
- Feature flag de domínio obrigatória.
- Nada de HEALTH no MVP 1.0.

## Alternativas consideradas

- Reutilizar CULTURE com pequenas variações: rejeitado por risco jurídico e semântico.
