# SPEC-XXX — Título

| Campo | Valor |
|:--|:--|
| ID | SPEC-XXX |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 (MVP) / 2 / 3 |
| Dono | Backend / Frontend / System Architect |
| Depende de | SPEC-... |
| Requisitos cobertos | RF-xxx, RB-xxx, RNF-xxx, AI-xxx |
| Última atualização | AAAA-MM-DD |

## 1. Objetivo

Uma frase: qual problema de negócio este módulo resolve e para quem.

## 2. Fora de escopo

O que **não** será feito aqui, com referência ao módulo/fase responsável.

## 3. Requisitos cobertos

| Requisito | Comportamento esperado | Verificação |
|:--|:--|:--|
| RF-xxx | ... | teste E2E ... |

## 4. Regras de negócio e invariantes

- RB-xxx: ...
- Invariante: ...

## 5. Modelo de dados

Coleções, campos, índices, invariantes de schema. Referenciar `DOMAIN_MODEL.md` quando já definido.

## 6. Contratos de API

| Método | Rota | Permissão | Request | Response | Erros |
|:--|:--|:--|:--|:--|:--|

## 7. Segurança e autorização

Capacidades, tenant, dados sensíveis, auditoria.

## 8. Processamento assíncrono

Filas, idempotência, retry, DLQ (quando aplicável).

## 9. Estados e transições

Máquina de estados e guardas (quando aplicável).

## 10. Critérios de aceite

```gherkin
Cenário: ...
  Dado ...
  Quando ...
  Então ...
```

## 11. Testes

- **Unitários:** ...
- **Integração:** ...
- **E2E:** ...
- **Segurança:** ...

## 12. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|

## 13. Definition of Done específico

- [ ] ...

## 14. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
