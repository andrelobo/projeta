# ADR-005 — LLM desacoplado de provedor

- **Status:** Aceito
- **Data:** 2026-11-05 (registro do PRD 1.0)
- **Decisores:** Product Owner / PRD 1.0
- **Origem:** PRD §21

## Contexto

Custo, qualidade e disponibilidade de modelos mudam rápido. O produto não pode depender de um único fornecedor nem reescrever a camada de IA a cada troca.

## Decisão

Definir uma **camada AI independente**, com interface de provedor, registry de modelos, prompts versionados, validação de saída, orçamento e fallback.

## Consequências

- Nenhum módulo de negócio chama SDK de fornecedor diretamente.
- Cada operação declara `taskId` (AI-001..AI-010) e versão de prompt.
- É possível trocar o provedor padrão sem alterar regras de negócio.
- Custo e latência precisam ser medidos por operação e por tenant.

## Alternativas consideradas

- Usar um único fornecedor e acoplar: rejeitado por risco estratégico.
- Orquestrador externo (ex.: gateway de LLM): possível, mas adiciona dependência; a interface interna resolve o essencial.
