# ADR-006 — CULTURE como domínio inicial

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

Os dois domínios têm fontes, regras e redes de relacionamento diferentes. Implementar ambos no MVP aumentaria escopo, risco e custo de validação.

## Decisão

Lançar o MVP com o domínio **CULTURE** funcional. HEALTH permanece previsto na arquitetura, sem fluxo funcional.

## Consequências

- Fontes, modelos e critérios do MVP são culturais.
- O campo `domain` é obrigatório e validado por registry desde o início.
- Nenhuma tela ou API do MVP presume que o usuário é apenas cultural; o modelo já aceita o segundo domínio.

## Alternativas consideradas

- Dois domínios simultâneos: rejeitado por escopo.
- Domínio genérico sem separação: rejeitado por risco de regras conflitantes.
