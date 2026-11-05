# Glossário — PROJETA

Termos canônicos usados em PRD, ADRs e especificações. Quando um termo divergir no código, o código está errado.

## Domínio e produto

| Termo | Definição |
|:--|:--|
| **Oportunidade** | Chamada pública ou privada que oferece financiamento, prêmio, bolsa ou apoio a projetos. Tem fonte verificável e ciclo de vida editorial próprio. |
| **Edital** | Documento oficial (PDF, página, anexo) que define regras, prazos, requisitos e critérios de uma oportunidade. |
| **Análise de edital** | Extração estruturada de um edital, com evidências por campo, confiança e status de validação. |
| **Requisito de elegibilidade** | Condição objetiva ou interpretável que o proponente deve cumprir (tipo de pessoa, localização, CNAE, documento, experiência etc.). |
| **Avaliação de elegibilidade** | Resultado determinístico da comparação entre perfil/projeto e requisitos, com justificativa por requisito. |
| **Projeto** | Conteúdo autoral do usuário (ideia, narrativa, metodologia, equipe, orçamento). Existe independentemente de edital. |
| **Proposta** | Instância do projeto vinculada a uma oportunidade específica. É o artefato que se versiona, congela e submete. |
| **Versão** | Snapshot imutável de uma proposta ou documento, com autor, data e motivo. |
| **Congelamento** | Ato de tornar uma versão imutável (ex.: ao marcar como `SUBMITTED`). |
| **Submissão** | Registro de que o usuário enviou a proposta ao financiador. O PROJETA **não** submete em nome do usuário. |
| **Radar** | Módulo de descoberta, busca, filtro, favoritos e alertas de oportunidades. |
| **Fonte** | Origem oficial ou institucional de uma oportunidade (portal, secretaria, fundação). |
| **Verificação** | Consulta à fonte para confirmar que a oportunidade existe, está aberta e mantém os dados publicados. |
| **Evidência** | Prova documental de um dado: página, trecho, documento, data e hash. |
| **Confiança** | Grau declarado de certeza de um dado extraído (alta, média, baixa), sempre acompanhado de evidência quando possível. |
| **Informação ausente** | Campo que o edital não informa ou que a extração não conseguiu determinar. Nunca deve ser preenchido por inferência. |

## Identidade e acesso

| Termo | Definição |
|:--|:--|
| **Usuário** | Pessoa física com conta na plataforma. |
| **Organização** | Pessoa jurídica ou coletivo formalizado que agrupa usuários e recursos. |
| **Membership** | Vínculo entre usuário e organização, com papel e estado. |
| **Papel (role)** | `OWNER`, `ADMIN`, `EDITOR` ou `VIEWER`. Define capacidades. |
| **Tenant** | Escopo de propriedade de recursos: um usuário ou uma organização. |
| **ownerType / ownerId** | Campos que definem o proprietário de um recurso (`USER` ou `ORGANIZATION`). |
| **Isolamento multi-tenant** | Garantia de que nenhum usuário acessa recurso de outro tenant sem autorização. |
| **IDOR** | *Insecure Direct Object Reference*: acesso a recurso alheio por troca de identificador. |

## Planos e cobrança

| Termo | Definição |
|:--|:--|
| **Plano** | Catálogo comercial (`FREE`, `INDIVIDUAL`, `PROFESSIONAL`, `STUDIO`) com limites e features. |
| **Entitlement** | Direito de uso derivado do plano: limite numérico, feature booleana ou quota. |
| **Quota** | Limite de uso por período (ex.: análises/mês, gerações/mês, armazenamento). |
| **Assinatura** | Vínculo cobrável entre tenant e plano, com status e ciclo de vida. |
| **Webhook** | Evento enviado pelo provedor de pagamento, validado por assinatura e idempotência. |
| **Idempotência** | Propriedade que garante que repetir a mesma operação não duplica efeitos. |

## IA

| Termo | Definição |
|:--|:--|
| **Camada AI** | Módulo independente que centraliza prompts, modelos, validação, RAG, custo e auditoria. |
| **Prompt template** | Template versionado com papel, instruções, formato de saída e delimitação de dados. |
| **Prompt injection** | Tentativa de um conteúdo externo (PDF, página) de alterar instruções do sistema. |
| **Context assembly** | Montagem controlada do contexto enviado ao modelo, com orçamento de tokens. |
| **RAG** | *Retrieval-Augmented Generation*: recuperação de trechos indexados para fundamentar a resposta. |
| **Chunk** | Fragmento indexado de documento, com página, seção, hash e metadados de tenant. |
| **Saída estruturada** | Resposta do modelo validada por schema (Zod), sem campos inventados. |
| **missingInformation** | Lista explícita de dados que o modelo não encontrou e que o usuário precisa fornecer. |
| **Custo de IA** | Tokens de entrada/saída, modelo, tempo e custo estimado por operação. |
| **Fallback** | Estratégia de degradação quando o provedor falha (modelo alternativo, fila, erro tratado). |

## Técnico

| Termo | Definição |
|:--|:--|
| **Monólito modular** | Uma aplicação implantável, com módulos isolados por domínio e fronteiras explícitas. |
| **Worker** | Processo que consome filas e executa tarefas assíncronas. |
| **Fila** | Canal BullMQ/Redis com retry, timeout, idempotência e DLQ. |
| **DLQ** | *Dead Letter Queue*: fila de mensagens que falharam definitivamente. |
| **Outbox** | Padrão de registro transacional de eventos para publicação confiável. |
| **Soft delete** | Marcação lógica de exclusão, preservando retenção e auditoria. |
| **Audit log** | Registro imutável de operações relevantes (quem, o quê, quando, de onde). |
| **Money** | Tipo de domínio para valores monetários (moeda + quantidade decimal), sem `float`. |
| **Decimal128** | Tipo BSON para decimais de alta precisão usado no MongoDB. |
| **Problem Details** | Formato padronizado de erro (`application/problem+json`, RFC 9457). |
| **Correlation ID** | Identificador que atravessa requisição, logs, jobs e chamadas externas. |
| **Effective status** | Situação derivada (ex.: `OPEN` que já passou do prazo passa a ser exibida como `CLOSED`), separada do status editorial persistido. |
| **Verification status** | Estado da confirmação da oportunidade junto à fonte: `VERIFIED`, `STALE`, `PENDING`, `FAILED`. |
