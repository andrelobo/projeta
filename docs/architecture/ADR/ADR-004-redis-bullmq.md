# ADR-004 — Redis + BullMQ para processamento assíncrono

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

Extração de PDF, OCR, análise por LLM, exportação de documentos e webhooks são operações lentas, falíveis e que não podem bloquear requisições HTTP.

## Decisão

Usar **Redis + BullMQ** para filas, com retry, timeout, idempotência, DLQ e workers dedicados.

## Consequências

- Redis passa a ser dependência crítica também para rate limit e quotas.
- Necessário monitorar profundidade de fila, falhas e DLQ.
- Jobs devem ser idempotentes e reportar progresso.
- Perda de Redis não pode corromper estado: jobs são reprocessáveis a partir do registro persistido.

## Alternativas consideradas

- Fila no próprio Mongo: simples, mas sem recursos maduros de agendamento/retry.
- Serviço gerenciado de fila: possível no futuro; adiar decisão.
