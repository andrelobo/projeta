# ADR-018 — Storage S3-compatible privado com URLs assinadas e varredura

- **Status:** Proposto — requer aprovação
- **Data:** 2026-11-05
- **Decisores:** System Architect
- **Relacionado:** RF-040, RF-090, RF-093, RNF-020

## Contexto

Documentos incluem RG/CPF, contratos, editais e propostas. Vazamento é risco jurídico e reputacional. O PRD exige storage privado e URL temporária.

## Decisão

Usar object storage compatível com S3, bucket privado, prefixo por tenant, upload com validação de MIME real/tamanho/hash e download por URL assinada de curta duração. Varredura antimalware antes de liberar.

## Justificativa

- S3-compatible permite MinIO local e troca de provedor sem mudar código.
- URL assinada evita exposição direta do bucket.
- Prefixo por tenant reduz risco de colisão e facilita retenção.

## Consequências

- Metadados no Mongo, binário no storage; nunca binário no banco.
- Documento `INFECTED` ou `FAILED` não é baixável.
- Backup do bucket é obrigatório (RNF-012).
- Provedor definitivo e região ficam em `OPEN-003`.

## Alternativas

- Armazenar em disco local: não escala e complica backup.
- URL pública com token: mais simples, porém pior em segurança.
