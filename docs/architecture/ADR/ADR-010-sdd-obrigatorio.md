# ADR-010 — Spec-Driven Development obrigatório

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

O desenvolvimento é assistido por agentes de IA. Sem especificação explícita, agentes tendem a inventar requisitos, simplificar regras e alterar arquitetura sem registro.

## Decisão

Adotar **SDD** como metodologia obrigatória: PRD → ADR → spec → implementação → testes → evidência. Nenhuma feature é implementada sem spec aprovada.

## Consequências

- Toda entrega referencia requisito e spec.
- Mudança de arquitetura exige ADR.
- Mudança de requisito exige atualização do PRD/spec e registro.
- O custo de documentação é compensado por previsibilidade e auditoria.

## Alternativas consideradas

- Desenvolvimento ad-hoc com revisão: rejeitado por risco de deriva em contexto de agentes.
