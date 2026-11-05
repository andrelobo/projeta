# ADR-017 — Proposta = ProjectOpportunity; versões escopadas por proposta

- **Status:** Aceito (2026-11-05, aprovado com o plano)
- **Data:** 2026-11-05
- **Decisores:** System Architect
- **Relacionado:** RF-066, RF-068, RF-069, RB-011

## Contexto

O PRD diz que um projeto pode se vincular a várias oportunidades "mantendo o histórico de cada proposta específica" e que versão submetida é imutável. Não fica explícito se a versão pertence ao projeto ou à proposta.

## Decisão

- `Project` é o conteúdo autoral-base.
- `ProjectOpportunity` é a **proposta**: projeto × oportunidade, com status próprio.
- `ProjectVersion` carrega `projectId` e `proposalId?`; quando há proposta, a versão é da proposta.
- `Budget` pertence à proposta quando existe; caso contrário, ao projeto.

## Justificativa

- Edital diferente exige versão, orçamento e requisitos diferentes.
- Submissão congela a proposta, não o projeto inteiro.
- Permite reutilizar o mesmo projeto em vários editais sem duplicar conteúdo-base.

## Consequências

- `project_versions` tem índice único por `(projectId, proposalId, versionNumber)`.
- Edição após congelamento cria nova versão da proposta.
- Projeto com proposta submetida não pode ser excluído.
- Exportação sempre referencia `versionId` (não "a última").

## Alternativas

- Versão só do projeto: impede históricos distintos por edital.
- Duplicar projeto por edital: perde reutilização e histórico.
