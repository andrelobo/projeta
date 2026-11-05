# SPEC-010 — Documentos e exportação

| Campo | Valor |
|:--|:--|
| ID | SPEC-010 |
| Versão | 0.1 |
| Status | Rascunho |
| Fase | 1 (núcleo) + 1 final (exportação) |
| Dono | Backend |
| Depende de | SPEC-000, SPEC-001, SPEC-002 |
| Requisitos cobertos | RF-090..094, RB-007, RNF-020 |
| Última atualização | 2026-11-05 |

## 1. Objetivo

Guardar documentos com segurança, classificar e vincular ao contexto correto, e gerar propostas em DOCX/PDF a partir de uma versão congelada — com autorização em todas as operações.

## 2. Fora de escopo

- Edição colaborativa de documentos.
- Assinatura digital (ICP-Brasil).
- Conversão de qualquer formato para PDF.
- Publicação pública de documentos.

## 3. Requisitos cobertos

| Requisito | Comportamento |
|:--|:--|
| RF-090 | storage privado |
| RF-091 | categorias de documento |
| RF-092 | metadados completos |
| RF-093 | autorização em todas as operações |
| RF-094 | DOCX/PDF a partir de modelos versionados |

## 4. Categorias

`PERSONAL`, `INSTITUTIONAL`, `PORTFOLIO`, `RESUME`, `PROOF`, `BUDGET`, `EDITAL`, `PROPOSAL`, `REPORT`.

## 5. Upload e armazenamento

1. Cliente solicita upload → API valida quota e gera URL pré-assinada (ou recebe multipart até 20 MB).
2. API valida **MIME real** por magic bytes, extensão e tamanho.
3. Calcula SHA-256; rejeita duplicata exata com aviso (não erro, salvo política).
4. Enfileira `document-processing` para varredura.
5. Documento só é baixável quando `scanStatus = CLEAN`.
6. Binário em bucket privado, prefixo `tenant/{ownerType}/{ownerId}/`.
7. Download por URL assinada com TTL de 5 minutos.

Limites configuráveis: tamanho por categoria, total por plano (SPEC-012).

## 6. Modelo de dados

### `documents`

| Campo | Tipo |
|:--|:--|
| `_id` | ObjectId |
| `owner` | OwnerRef |
| `category` | enum |
| `name` | string |
| `mime` | string |
| `size` | number |
| `hash` | string (SHA-256) |
| `storageKey` | string |
| `scanStatus` | `PENDING / CLEAN / INFECTED / FAILED` |
| `source` | `UPLOAD / GENERATED` |
| `linkedTo` | `{ type: 'PROFILE' / 'ORGANIZATION' / 'PROJECT' / 'PROPOSAL' / 'BUDGET' / 'ANALYSIS', id }?` |
| `retentionUntil` | Date? |
| `createdBy` | ObjectId |
| `createdAt` / `deletedAt` | Date |

### `document_versions`

`{ _id, documentId, version, storageKey, hash, size, createdBy, createdAt }`

### `document_templates` (derivada — RF-094)

`{ _id, code, name, version, format: DOCX|PDF, bodyRef, status: DRAFT|ACTIVE|RETIRED, createdAt }`

> Coleção adicional necessária para atender "modelos versionados" (RF-094). Registrada como decisão derivada em `CONSISTENCY_REPORT`.

## 7. Contratos de API

| Método | Rota | Permissão | Request | Response |
|:--|:--|:--|:--|:--|
| POST | `/documents` | autenticado + quota | multipart ou `{ name, category, mime, size, linkedTo?, contentBase64? }` | `201` documento |
| GET | `/documents` | autenticado | `category`, `linkedTo`, paginação | lista |
| GET | `/documents/{documentId}` | dono/VIEWER+ | — | metadados |
| DELETE | `/documents/{documentId}` | dono/ADMIN+ | — | `204` soft delete |
| GET | `/documents/{documentId}/download` | dono/VIEWER+ | — | `302` URL assinada TTL 5 min |
| GET | `/documents/{documentId}/versions` | dono/VIEWER+ | — | versões |
| POST | `/exports` | EDITOR+ + quota | `{ versionId, format: DOCX / PDF, templateCode? }` | `202 { jobId }` |
| GET | `/exports/{jobId}` | dono | — | status + documentId |

### Erros

| HTTP | code | Quando |
|:--|:--|:--|
| 413 | `FILE_TOO_LARGE` | acima do limite |
| 415 | `UNSUPPORTED_FILE_TYPE` | MIME real não permitido |
| 422 | `DOCUMENT_INFECTED` | varredura positiva |
| 403 | `DOCUMENT_FORBIDDEN` | sem permissão |
| 409 | `DOCUMENT_LINKED` | excluir documento vinculado a proposta submetida |
| 422 | `VERSION_NOT_FROZEN` | exportar versão não congelada (quando exigido) |
| 502 | `RENDER_FAILED` | falha na geração |

## 8. Exportação

- Entrada: `versionId` (obrigatório), formato, template.
- Renderização em worker (`document-export`).
- DOCX: biblioteca `docx`; PDF: conversão por serviço dedicado (decisão `OPEN-037`).
- Saída vira `Document` (`source = GENERATED`, `category = PROPOSAL`) e é vinculada à versão.
- Nome do arquivo: `projeta_{slug}_{version}_{timestamp}.{ext}`.
- Exportação idempotente por `(versionId, format, templateVersion)`.
- Toda exportação é auditada.

## 9. Segurança

- Bucket sem acesso público; política de menor privilégio.
- URL assinada curta; `Content-Disposition: attachment`.
- Varredura antimalware obrigatória.
- Nenhum documento de outro tenant acessível por troca de id (RB-007).
- Downloads registrados em auditoria (quem, quando, IP).
- Retenção: documentos de proposta submetida seguem política `OPEN-003/OPEN-023`.

## 10. Critérios de aceite

```gherkin
Cenário: download exige autorização
  Dado um documento do tenant A
  Quando o usuário do tenant B solicita download
  Então a resposta é 404

Cenário: documento infectado não é servido
  Dado um upload cuja varredura retorna INFECTED
  Quando o download é solicitado
  Então a resposta é 422 DOCUMENT_INFECTED
  E o binário não é disponibilizado

Cenário: exportação usa versão explícita
  Dado duas versões de uma proposta
  Quando o usuário exporta a versão 1
  Então o DOCX contém o conteúdo da versão 1
  E não o rascunho atual

Cenário: tipo real é validado
  Dado um executável renomeado para .pdf
  Quando o upload é enviado
  Então a resposta é 415
```

## 11. Testes

- **Unitários:** validação de MIME, hash, nome de arquivo, autorização.
- **Integração:** upload → scan → download; versões; exportação DOCX/PDF.
- **E2E:** anexar currículo, exportar proposta, baixar arquivo.
- **Segurança:** path traversal, IDOR, URL expirada, upload malicioso, zip bomb.

## 12. Riscos e decisões abertas

| ID | Questão | Impacto | Encaminhamento |
|:--|:--|:--|:--|
| OPEN-003 | storage definitivo e região | alto | ADR |
| OPEN-037 | renderização PDF (LibreOffice/Gotenberg/serviço) | médio | ADR |
| OPEN-023 | retenção de documentos e PDFs | alto | LGPD |
| OPEN-038 | limite de armazenamento por plano | médio | SPEC-012 |

## 13. Definition of Done específico

- [ ] Nenhum download sem autorização.
- [ ] Scan obrigatório antes de disponibilizar.
- [ ] Exportação determinística por versão.
- [ ] Auditoria de download.
- [ ] Teste de IDOR cross-tenant.

## 14. Changelog

| Data | Versão | Autor | Alteração |
|:--|:--|:--|:--|
| 2026-11-05 | 0.1 | System Architect | criação |
