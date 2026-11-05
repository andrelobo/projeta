# ADR-001 — Monólito modular NestJS

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

O produto tem 13 módulos funcionais, time pequeno e estágio de validação de mercado. Microsserviços adicionariam custo operacional (deploy, observabilidade, consistência distribuída) sem benefício comprovado.

## Decisão

Construir a API como **monólito modular** em NestJS, com módulos isolados por contexto, serviços públicos por módulo e proibição de acesso cruzado a repositórios.

## Consequências

- Um único deploy de API; workers em processo separado.
- Fronteiras verificadas por revisão e, futuramente, lint de dependências.
- Extração futura de um módulo para serviço é possível, mas exige ADR.
- Transações locais no MongoDB são suficientes para o MVP.

## Alternativas consideradas

- Microsserviços desde o início: rejeitado por custo e ausência de escala que justifique.
- Monólito sem modularidade: rejeitado por risco de acoplamento e dificuldade de extração futura.
