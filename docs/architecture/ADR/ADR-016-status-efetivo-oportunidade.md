# ADR-016 — Status editorial persistido + status efetivo derivado + verificação

- **Status:** Proposto — requer aprovação
- **Data:** 2026-11-05
- **Decisores:** System Architect
- **Relacionado:** RF-033, RF-038, RB-019

## Contexto

O PRD tem um enum de status (`OPEN`, `CLOSED` etc.) e, ao mesmo tempo, exige que `OPEN` não permaneça após o prazo sem nova verificação e que a UI mostre "confirmação pendente". Um único campo não expressa as duas coisas.

## Decisão

Separar três conceitos:

1. `status` — estado editorial persistido (curadoria).
2. `verificationStatus` — `VERIFIED|PENDING|STALE|FAILED`, com `lastVerifiedAt`.
3. `effectiveStatus` — derivado em leitura: se `status ∈ {OPEN, UPCOMING}` e `closingDate < now`, exibe `CLOSED`.

## Justificativa

- Evita job que "reabre" oportunidade por engano.
- Permite exibir "Encerrado — confirmação pendente" sem mentir.
- Mantém histórico editorial íntegro.

## Consequências

- Toda listagem/API retorna `effectiveStatus` e `confirmationPending`.
- Nenhuma tela pode exibir `OPEN` com prazo vencido.
- Um job diário marca `CLOSED` editorial com motivo `DATE_PASSED` e `verificationStatus = PENDING`.
- Filtros usam `effectiveStatus`, não `status` isolado.

## Alternativas

- Um único campo: não cobre RF-038/RB-019 sem estados artificiais.
- Apenas derivar tudo: perde curadoria (suspenso, cancelado).
