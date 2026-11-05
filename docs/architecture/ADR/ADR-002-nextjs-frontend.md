# ADR-002 — Next.js no frontend

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

A interface exige dashboard, formulários densos, listagens com filtros, estados de processamento e boa experiência em desktop e mobile. A equipe já domina React/TypeScript.

## Decisão

Usar **Next.js + TypeScript**, com App Router, renderização no servidor para páginas autenticadas e componentes cliente para interações.

## Consequências

- Um único framework cobre páginas públicas e área logada.
- Autenticação por cookie `httpOnly` conversa bem com SSR.
- Necessário cuidado com cache de dados privados e vazamento entre requisições.
- O frontend **não** é fonte de autorização: apenas reflete capacidades.

## Alternativas consideradas

- SPA Vite + React: viável, mas perde SSR e convenções de rota.
- Remix: não há ganho decisivo para o escopo.
