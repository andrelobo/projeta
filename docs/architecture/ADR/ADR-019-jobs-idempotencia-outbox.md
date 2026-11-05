# ADR-019 — Jobs idempotentes, outbox transacional e DLQ

- **Status:** Proposto — requer aprovação
- **Data:** 2026-11-05
- **Decisores:** System Architect
- **Relacionado:** ADR-004, RNF-009, RNF-019, RB-017

## Contexto

Filas entregam pelo menos uma vez. Sem idempotência, um retry duplica cobrança, exportação ou análise. Sem outbox, um evento pode ser perdido entre o commit e a publicação.

## Decisão

- Todo job declara `idempotencyKey` persistida e verificada antes de executar.
- Eventos de negócio relevantes usam **outbox** no mesmo commit do estado.
- Retry com backoff, `attempts`, timeout e DLQ; reprocessamento manual auditado.
- Webhooks de billing usam `providerEventId` único.

## Justificativa

- Evita efeitos duplicados em dinheiro, notificações e documentos.
- Preserva consistência entre banco e fila.
- Facilita diagnóstico e reprocessamento.

## Consequências

- Coleção `outbox` com índice em `status, availableAt`.
- Publisher dedicado no worker.
- Testes de concorrência e de replay obrigatórios.

## Alternativas

- Publicar direto na fila após commit: janela de perda.
- Exactly-once do broker: não existe de forma confiável; idempotência na aplicação resolve.
