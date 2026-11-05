# ADR-025 — Licença Apache-2.0

- **Status:** Aceito
- **Data:** 2026-11-05
- **Decisor:** Product Owner (delegou a escolha; aprovada)
- **Relacionado:** ADR-021 (repositórios públicos), OPEN-047

## Contexto

Os três repositórios são públicos e o produto é um SaaS comercial. É preciso uma licença que dê clareza jurídica, permita uso e contribuição por terceiros e não contamine o código proprietário com copyleft. Sem licença, o código público fica “all rights reserved”, o que gera ambiguidade para contribuidores e usuários.

## Decisão

Adotar **Apache License 2.0** em `projeta`, `projeta-web` e `projeta-api`.

- Licença permissiva, com concessão explícita de patentes (seção 3).
- Permite uso comercial, modificação e distribuição, inclusive por concorrentes.
- Exige preservação de avisos e do `LICENSE`; não exige abrir derivados.
- Marca, nome e identidade visual **não** são licenciados (seção 6).

## Consequências

- Qualquer pessoa pode hospedar uma cópia do PROJETA; a vantagem competitiva passa a ser **dados, curadoria, operação, marca e suporte**, não o código.
- Contribuições externas entram sob a mesma licença (seção 5), sem CLA adicional.
- Documentação também é licenciada sob Apache-2.0.
- Se o objetivo mudar para impedir oferta concorrente do SaaS, será necessário um novo ADR (AGPL-3.0 ou BSL) **antes** de aceitar contribuições relevantes — trocar licença depois é juridicamente complexo.

## Alternativas consideradas

| Licença | Por que não |
|:--|:--|
| MIT | Mais simples, porém sem concessão explícita de patentes. |
| AGPL-3.0 | Protege o SaaS contra cópia hospedada, mas afasta adoção e impõe obrigações fortes. |
| BSL 1.1 | Source-available, restringe uso em produção; não é OSI e adiciona complexidade. |
| Proprietária | Contradiz a decisão de repositórios públicos. |
