# PROJETA — PRD 1.0

> **Emendas aprovadas:** este PRD é a baseline. Mudanças do Product Owner estão registradas em [`PRD_ERRATA.md`](PRD_ERRATA.md) e prevalecem em caso de conflito. Em especial, a **Emenda 01** substitui §9.1 (monorepo) por repositórios separados no GitHub.

**Documento de produto · Spec-Driven Development**

| Campo | Valor |
|:--|:--|
| Nome | PROJETA |
| Categoria | SaaS — Project Intelligence & Management Platform |
| Versão | 1.0.0 |
| Status | Baseline de produto para desenvolvimento |
| Mercado | Brasil |
| Idioma inicial | Português brasileiro (pt-BR) |
| Modelo de negócio | Freemium + assinatura recorrente |
| Arquitetura | Modular Monolith + Background Workers |
| Metodologia | Spec-Driven Development (SDD) |
| Ambiente de engenharia | DeepSeek Harness |
| Stack principal | Next.js, TypeScript, NestJS, MongoDB, Redis e LLM APIs |

> **Objetivo do documento:** entregar uma especificação completa, preparada para ser utilizada por agentes de desenvolvimento como contexto de engenharia e implementação.
>
> **Premissa central:** o agente não deve inventar requisitos, simplificar regras de negócio ou modificar decisões arquiteturais sem registrar e justificar a mudança.
>
> **Nota de transcrição (2026-11-05):** este arquivo reproduz o PRD 1.0 fornecido pelo product owner. Os caminhos de API que apareceram sem placeholder na fonte foram normalizados para `{id}` / `{organizationId}` / `{projectId}` / `{budgetId}` / `{documentId}` / `{assessmentId}` / `{analysisId}`, sem alteração semântica. Nenhum requisito foi acrescentado ou removido. Ver `docs/validation/CONSISTENCY_REPORT.md`.

---

## 1. VISÃO DO PRODUTO

### 1.1 Resumo executivo

PROJETA é uma plataforma SaaS que permite a profissionais independentes, artistas, produtores culturais, consultores, coletivos e organizações encontrar oportunidades de financiamento, desenvolver projetos, preparar propostas, organizar documentação e gerenciar a execução dos projetos aprovados.

O produto atenderá dois domínios de negócio independentes:

- **CULTURE:** projetos culturais, artísticos, patrimoniais e criativos.
- **HEALTH:** projetos de promoção da saúde, prevenção, educação em saúde e iniciativas comunitárias.

Os domínios compartilham infraestrutura, identidade, documentos, projetos e ferramentas de produtividade, mas possuem regras de elegibilidade, fontes, modelos, validações e fluxos especializados.

A primeira versão comercial será concentrada no domínio **CULTURE**.

O domínio **HEALTH** deverá estar previsto arquiteturalmente, mas não será implementado funcionalmente no MVP 1.0.

### 1.2 Problema

Profissionais e organizações enfrentam dificuldades para:

1. Encontrar oportunidades compatíveis com seu perfil.
2. Identificar requisitos de participação.
3. Interpretar editais e regulamentos.
4. Desenvolver propostas técnicas.
5. Construir orçamentos e cronogramas.
6. Organizar documentação.
7. Preparar submissões.
8. Acompanhar prazos e resultados.
9. Administrar projetos aprovados.
10. Organizar prestação de contas.

As informações estão distribuídas em diferentes portais, documentos, páginas institucionais e plataformas.

PROJETA deve reduzir essa fragmentação.

### 1.3 Proposta de valor

Transformar oportunidades em projetos estruturados, propostas preparadas e operações administráveis.

### 1.4 Visão de longo prazo

Ser uma plataforma brasileira de referência para inteligência, desenvolvimento e gestão de projetos financiados.

### 1.5 Objetivos estratégicos

- Democratizar o acesso a oportunidades.
- Reduzir o tempo de elaboração de projetos.
- Melhorar a qualidade técnica das propostas.
- Diminuir erros documentais.
- Aumentar a capacidade de organização dos profissionais.
- Criar receita recorrente por assinatura.
- Construir uma base estruturada de conhecimento sobre oportunidades.

---

## 2. OBJETIVOS E NÃO OBJETIVOS

### 2.1 Objetivos do MVP

O MVP deve permitir que um usuário:

- Crie sua conta.
- Cadastre seu perfil profissional.
- Consulte oportunidades culturais.
- Pesquise e filtre oportunidades.
- Salve oportunidades.
- Importe um edital em PDF.
- Obtenha uma análise estruturada do edital.
- Verifique compatibilidade com seu perfil.
- Crie um projeto a partir de uma ideia.
- Utilize IA para desenvolver uma proposta.
- Edite e versiona o conteúdo.
- Exporte documentos.
- Organize seus projetos.
- Gerencie sua assinatura.

### 2.2 Não objetivos do MVP

Não implementar nesta primeira versão:

- Marketplace de profissionais.
- Marketplace de patrocinadores.
- Rede social.
- Sistema de avaliação pública de projetos.
- Submissão automática em portais governamentais.
- Integração automática com todos os portais de fomento.
- Prestação de contas avançada.
- Gestão de pagamentos de projetos financiados.
- Domínio HEALTH funcional.
- Aplicativo mobile nativo.
- Treinamento de modelos próprios.
- Garantia ou previsão de aprovação de propostas.

### 2.3 Princípio de escopo

Nenhuma funcionalidade fora do MVP pode ser adicionada sem avaliação de impacto em prazo, custo, arquitetura e prioridade.

---

## 3. PÚBLICO-ALVO E PERSONAS

### 3.1 Persona P01 — Artista independente

Características:

- Trabalha individualmente.
- Possui portfólio.
- Participa de editais ocasionalmente.
- Tem dificuldade com documentos e orçamento.
- Busca oportunidades compatíveis.

Necessidades:

- Encontrar editais.
- Entender requisitos.
- Desenvolver propostas.
- Organizar documentos.

### 3.2 Persona P02 — Produtor cultural

Características:

- Administra vários projetos.
- Trabalha com equipes.
- Participa de diferentes chamadas públicas.
- Precisa controlar prazos e orçamentos.

Necessidades:

- Radar de oportunidades.
- Múltiplos projetos.
- Histórico de propostas.
- Gestão documental.
- Produtividade.

Persona prioritária para monetização inicial.

### 3.3 Persona P03 — Consultor de projetos

Características:

- Desenvolve propostas para clientes.
- Trabalha com múltiplas organizações.
- Precisa reutilizar modelos e informações.
- Administra entregas simultâneas.

Necessidades:

- Gestão de portfólio.
- Múltiplos perfis organizacionais.
- Versionamento.
- Exportação.
- Colaboração.

### 3.4 Persona P04 — Organização cultural

Características:

- Associação, coletivo formalizado ou instituição.
- Possui representantes e colaboradores.
- Trabalha com financiamento público e privado.

Necessidades:

- Documentação institucional.
- Perfis de equipe.
- Gestão de projetos.
- Controle de acesso.

### 3.5 Persona P05 — Instituição de saúde

Persona futura do domínio HEALTH.

Não participa do escopo funcional do MVP 1.0.

---

## 4. PRINCÍPIOS DE PRODUTO

- **P01.** A fonte oficial prevalece sobre qualquer interpretação da IA.
- **P02.** Toda oportunidade precisa ter proveniência identificável.
- **P03.** O sistema não pode apresentar oportunidades encerradas como abertas.
- **P04.** O sistema não pode inventar requisitos, valores, datas ou critérios.
- **P05.** A IA deve identificar informações ausentes.
- **P06.** O usuário mantém controle sobre o conteúdo produzido.
- **P07.** Nenhum documento será submetido automaticamente sem autorização explícita.
- **P08.** Cultura e Saúde são domínios independentes.
- **P09.** Os dados de cada organização devem permanecer isolados.
- **P10.** Funcionalidades pagas devem respeitar limites de utilização.
- **P11.** Toda operação relevante precisa ter rastreabilidade.
- **P12.** A plataforma não promete aprovação de projetos.

---

## 5. ESCOPO FUNCIONAL DO MVP

### MÓDULO M01 — IDENTIDADE E AUTENTICAÇÃO

**RF-001 — Cadastro.** O sistema deve permitir cadastro utilizando:

- Nome completo.
- E-mail.
- Senha.
- Aceite dos termos.
- Aceite da política de privacidade.

Critérios de aceite:

- E-mail único.
- Senha armazenada com hash seguro.
- Validação dos dados.
- Confirmação de cadastro.
- Tratamento de erros.

**RF-002 — Login.** O sistema deve permitir autenticação segura.

**RF-003 — Recuperação de senha.** O usuário deve conseguir solicitar recuperação por e-mail.

**RF-004 — Perfil de usuário.** O usuário deve poder consultar e atualizar seus dados.

**RF-005 — Exclusão de conta.** O usuário deve poder solicitar exclusão da conta, respeitando obrigações legais de retenção.

### MÓDULO M02 — ORGANIZAÇÕES

**RF-010 — Cadastro de organização.** Permitir cadastro de:

- Nome.
- Nome fantasia.
- CNPJ, quando aplicável.
- Natureza jurídica.
- Endereço.
- Área de atuação.
- Responsável.
- Documentação institucional.

**RF-011 — Associação de usuários.** Permitir vincular usuários a organizações.

**RF-012 — Permissões.** Papéis iniciais:

- OWNER
- ADMIN
- EDITOR
- VIEWER

**RF-013 — Isolamento.** Nenhum usuário pode acessar dados de organização sem associação e permissão válida.

### MÓDULO M03 — PERFIL PROFISSIONAL

**RF-020 — Perfil.** Permitir cadastrar:

- Áreas de atuação.
- Formação.
- Experiência.
- Portfólio.
- Projetos anteriores.
- Habilidades.
- Localização.
- Abrangência territorial.
- Tipo de proponente.

**RF-021 — Documentação.** Permitir associar documentos ao perfil.

**RF-022 — Reutilização.** Dados do perfil poderão preencher informações em projetos, mediante confirmação do usuário.

**RF-023 — Atualização.** Alterações no perfil não podem modificar retroativamente versões já congeladas de propostas submetidas.

### MÓDULO M04 — RADAR DE OPORTUNIDADES

**RF-030 — Cadastro de oportunidades.** O sistema deve manter oportunidades estruturadas.

Campos:

- title
- slug
- domain
- description
- organizer
- sourceUrl
- sourceName
- publicationDate
- openingDate
- closingDate
- minimumAmount
- maximumAmount
- currency
- geographicScope
- eligibleApplicantTypes
- thematicAreas
- requirements
- requiredDocuments
- evaluationCriteria
- status
- lastVerifiedAt
- verificationStatus

**RF-031 — Busca.** Permitir pesquisa por título, descrição, organizador e palavras-chave.

**RF-032 — Filtros.** Filtros mínimos:

- Área.
- Estado.
- Município.
- Abrangência.
- Valor.
- Modalidade.
- Tipo de proponente.
- Prazo.
- Situação.

**RF-033 — Situação.** Estados:

- DRAFT
- DISCOVERED
- UNDER_REVIEW
- OPEN
- UPCOMING
- CLOSED
- SUSPENDED
- CANCELLED
- ARCHIVED

**RF-034 — Verificação de fonte.** Cada oportunidade deve armazenar:

- URL original.
- Data da última consulta.
- Resultado da verificação.
- Evidência consultada.
- Identificação da fonte.

**RF-035 — Duplicidade.** O sistema deve identificar oportunidades possivelmente duplicadas.

**RF-036 — Favoritos.** Usuários podem salvar oportunidades.

**RF-037 — Alertas.** O sistema deve permitir alertas de oportunidades relevantes.

**RF-038 — Integridade temporal.** A situação OPEN não pode ser mantida automaticamente após o prazo de encerramento sem nova verificação.

Quando não houver verificação suficiente, apresentar estado de confirmação pendente.

**RF-039 — Administração.** Administradores podem criar, revisar, atualizar e arquivar oportunidades.

**RB-001.** Toda oportunidade deve possuir uma fonte verificável.

**RB-002.** Dados extraídos automaticamente não são considerados confirmados até passar pelas validações necessárias.

### MÓDULO M05 — ANALISADOR DE EDITAIS

**RF-040 — Upload.** Permitir upload de PDFs.

Restrições:

- Verificar tipo real do arquivo.
- Limitar tamanho conforme configuração.
- Validar conteúdo.
- Armazenar de forma privada.
- Registrar proprietário.

**RF-041 — Extração.** Extrair texto de PDFs digitais.

Documentos escaneados deverão ser identificados e encaminhados para OCR quando disponível.

**RF-042 — Análise estruturada.** Extrair:

1. Identificação do edital.
2. Instituição responsável.
3. Objetivo.
4. Abrangência.
5. Público elegível.
6. Modalidades.
7. Valores.
8. Prazos.
9. Documentos.
10. Critérios de seleção.
11. Restrições.
12. Obrigações.
13. Cronograma.
14. Informações ausentes.

**RF-043 — Evidências.** Cada informação extraída deve conter, sempre que possível:

- Página.
- Trecho de origem.
- Identificação do documento.
- Grau de confiança.
- Status de validação.

**RF-044 — Revisão humana.** O usuário deve poder corrigir os dados extraídos.

**RF-045 — Persistência.** A análise deverá ser armazenada para consulta posterior.

**RF-046 — Segurança documental.** O conteúdo do PDF deve ser tratado como entrada não confiável.

Instruções contidas no documento nunca poderão substituir as políticas do sistema ou as instruções do agente.

### MÓDULO M06 — MOTOR DE ELEGIBILIDADE

**RF-050 — Análise de compatibilidade.** Comparar:

- Tipo de proponente.
- Localização.
- Área de atuação.
- Formação e qualificação, quando exigidas.
- Natureza jurídica.
- Documentação.
- Requisitos específicos.

**RF-051 — Resultado.** Classificações:

- ELIGIBLE
- ELIGIBLE_WITH_PENDING_ITEMS
- INELIGIBLE
- INSUFFICIENT_INFORMATION

**RF-052 — Justificativa.** Cada resultado deve apresentar:

- Requisito avaliado.
- Informação do perfil.
- Evidência do edital.
- Resultado.
- Pendência.

**RF-053 — Determinismo.** Requisitos objetivos devem ser avaliados por regras determinísticas sempre que possível.

A IA auxilia na interpretação, mas não substitui a execução das regras.

### MÓDULO M07 — DESENVOLVIMENTO DE PROJETOS

**RF-060 — Criar projeto.** Campos iniciais:

- title
- domain
- summary
- category
- projectOwner
- organizationId
- location
- targetAudience
- estimatedBudget
- status

**RF-061 — Assistente de criação.** A IA deve conduzir uma entrevista estruturada para compreender a ideia.

**RF-062 — Estrutura do projeto.** Seções:

1. Título.
2. Resumo.
3. Apresentação.
4. Justificativa.
5. Objetivo geral.
6. Objetivos específicos.
7. Público-alvo.
8. Metodologia.
9. Plano de atividades.
10. Metas.
11. Indicadores.
12. Cronograma.
13. Equipe.
14. Orçamento.
15. Plano de comunicação.
16. Acessibilidade.
17. Democratização de acesso.
18. Sustentabilidade.
19. Resultados esperados.

As seções deverão ser configuráveis conforme o edital.

**RF-063 — Contextualização.** A IA deve utilizar:

- Dados fornecidos pelo usuário.
- Perfil profissional.
- Documentos autorizados.
- Edital selecionado.
- Modelos aplicáveis.

**RF-064 — Informações ausentes.** A IA não deve inventar:

- Participantes.
- Resultados anteriores.
- Currículos.
- Contratos.
- Parcerias.
- Valores.
- Dados estatísticos.

Deve solicitar complementação.

**RF-065 — Edição.** O usuário pode editar todas as seções.

**RF-066 — Versionamento.** Cada versão deve possuir:

- Identificador.
- Número.
- Data.
- Autor.
- Alterações.
- Conteúdo preservado.

**RF-067 — Histórico.** Permitir consultar versões anteriores.

**RF-068 — Vinculação.** Um projeto poderá ser vinculado a uma ou mais oportunidades, mantendo o histórico de cada proposta específica.

**RF-069 — Congelamento.** Uma versão marcada como SUBMITTED deve permanecer imutável. Correções posteriores deverão criar nova versão.

### MÓDULO M08 — ORÇAMENTO

**RF-070 — Categorias.** Permitir categorias configuráveis.

Exemplos:

- Recursos humanos.
- Serviços.
- Materiais.
- Equipamentos.
- Transporte.
- Alimentação.
- Comunicação.
- Acessibilidade.
- Infraestrutura.

**RF-071 — Itens.** Cada item deverá possuir:

- Descrição.
- Categoria.
- Quantidade.
- Unidade.
- Valor unitário.
- Valor total.
- Justificativa.
- Etapa vinculada.

**RF-072 — Cálculo.** Valor total = quantidade × valor unitário.

Somatórios devem ser realizados pelo backend utilizando precisão decimal adequada.

**RF-073 — Limites.** Comparar o orçamento com limites do edital.

**RF-074 — Exportação.** Exportar orçamento em XLSX e PDF.

**RF-075 — Validação.** Identificar:

- Valores ausentes.
- Totais inconsistentes.
- Categorias não permitidas.
- Limites excedidos.
- Itens sem justificativa.

### MÓDULO M09 — CRONOGRAMA

**RF-080 — Atividades.** Cadastrar atividades, responsáveis, datas e dependências.

**RF-081 — Etapas.** Organizar atividades por etapas.

**RF-082 — Validação.** Identificar datas inconsistentes e prazos incompatíveis com o edital.

**RF-083 — Visualização.** Disponibilizar visualização em lista e calendário.

### MÓDULO M10 — DOCUMENTOS

**RF-090 — Armazenamento.** Armazenar documentos em storage privado.

**RF-091 — Classificação.** Categorias:

- Pessoal.
- Institucional.
- Portfólio.
- Currículo.
- Comprovantes.
- Orçamentos.
- Editais.
- Propostas.
- Relatórios.

**RF-092 — Metadados.** Registrar:

- Proprietário.
- Organização.
- Tipo.
- Nome.
- Tamanho.
- MIME.
- Hash.
- Data de envio.
- Versão.

**RF-093 — Permissões.** Validar autorização em todas as operações.

**RF-094 — Exportação de proposta.** Gerar DOCX e PDF a partir de modelos versionados.

### MÓDULO M11 — WORKSPACE E DASHBOARD

**RF-100 — Dashboard.** Exibir:

- Oportunidades relevantes.
- Prazos próximos.
- Projetos em andamento.
- Projetos submetidos.
- Pendências.
- Atividades recentes.

**RF-101 — Gestão de projetos.** Visualizar projetos por status.

Estados:

- IDEA
- DRAFT
- IN_DEVELOPMENT
- UNDER_REVIEW
- READY_TO_SUBMIT
- SUBMITTED
- APPROVED
- REJECTED
- IN_EXECUTION
- COMPLETED
- ARCHIVED

**RF-102 — Atividades.** Permitir criar tarefas vinculadas aos projetos.

**RF-103 — Prazos.** Gerar alertas para vencimentos.

### MÓDULO M12 — ASSINATURAS E BILLING

**RF-110 — Planos.** Planos iniciais:

**FREE**
- Consulta de oportunidades.
- Perfil básico.
- Projetos limitados.
- Análises limitadas.

**INDIVIDUAL**
- Radar personalizado.
- Assistência de IA.
- Análises adicionais.
- Exportações.

**PROFESSIONAL**
- Maior volume de análises.
- Mais projetos.
- Orçamento.
- Cronograma.
- Gestão documental.

**STUDIO**
- Equipes.
- Organizações.
- Portfólio ampliado.
- Colaboração.

**RF-111 — Assinatura.** Permitir contratar e gerenciar planos.

**RF-112 — Limites.** Controlar:

- Projetos ativos.
- Análises de edital.
- Gerações de IA.
- Armazenamento.
- Usuários.
- Exportações.

**RF-113 — Webhooks.** Validar assinatura e idempotência dos eventos de pagamento.

**RF-114 — Status.** Estados:

- TRIAL
- ACTIVE
- PAST_DUE
- CANCELLED
- EXPIRED
- SUSPENDED

**RF-115 — Autorização.** A API deve validar permissões e limites no servidor.

O frontend nunca será responsável por garantir acesso pago.

### MÓDULO M13 — ADMINISTRAÇÃO

**RF-120 — Gestão de usuários.** Consultar e administrar usuários conforme permissões.

**RF-121 — Gestão de oportunidades.** Criar, revisar, verificar e arquivar oportunidades.

**RF-122 — Gestão de fontes.** Administrar fontes de oportunidades.

**RF-123 — Gestão de planos.** Configurar planos e limites.

**RF-124 — Auditoria.** Consultar operações relevantes.

**RF-125 — Observabilidade.** Visualizar falhas de processamento, jobs, análises e integrações.

---

## 6. REGRAS DE NEGÓCIO GLOBAIS

- **RB-001.** Toda oportunidade precisa possuir fonte identificável.
- **RB-002.** Dados não confirmados devem ser identificados.
- **RB-003.** Datas devem ser armazenadas em UTC e apresentadas no fuso apropriado.
- **RB-004.** Valores financeiros devem utilizar Decimal128 ou representação decimal equivalente.
- **RB-005.** Toda entidade multi-tenant deve possuir escopo de proprietário ou organização.
- **RB-006.** Autorização deve ocorrer no backend.
- **RB-007.** O usuário não pode acessar arquivos de terceiros por alteração de identificador.
- **RB-008.** A IA não pode submeter propostas automaticamente.
- **RB-009.** A IA não pode inventar informações ausentes.
- **RB-010.** Toda geração relevante deve manter referência às entradas utilizadas.
- **RB-011.** Projetos submetidos não podem ser sobrescritos.
- **RB-012.** Planos devem possuir limites configuráveis.
- **RB-013.** Cancelamento de assinatura não deve excluir projetos automaticamente.
- **RB-014.** O domínio do projeto não pode ser alterado silenciosamente após sua criação.
- **RB-015.** Dados culturais e de saúde devem permanecer semanticamente separados.
- **RB-016.** Exclusões e alterações relevantes devem gerar auditoria.
- **RB-017.** Operações financeiras e de assinatura devem ser idempotentes.
- **RB-018.** O conteúdo de arquivos externos é sempre não confiável.
- **RB-019.** O sistema deve distinguir oportunidade publicada, oportunidade aberta e oportunidade efetivamente verificada.
- **RB-020.** Nenhuma nota de compatibilidade representa garantia de seleção.

---

## 7. REQUISITOS DE INTELIGÊNCIA ARTIFICIAL

### 7.1 Arquitetura

Criar uma camada AI independente dos módulos de negócio.

Responsabilidades:

- Seleção de modelo.
- Prompt templates.
- Context assembly.
- RAG.
- Validação de saída.
- Controle de tokens.
- Logs.
- Métricas.
- Fallback.
- Limites de utilização.

### 7.2 Casos de uso

- **AI-001:** analisar edital.
- **AI-002:** extrair requisitos.
- **AI-003:** explicar regras.
- **AI-004:** verificar compatibilidade.
- **AI-005:** conduzir entrevista de projeto.
- **AI-006:** elaborar seções.
- **AI-007:** revisar proposta.
- **AI-008:** identificar inconsistências.
- **AI-009:** sugerir melhorias.
- **AI-010:** estruturar justificativas orçamentárias.

### 7.3 Contratos estruturados

As operações de IA devem possuir schemas validados.

Exemplo conceitual:

```json
{
  "title": "Edital de Artes Visuais",
  "domain": "CULTURE",
  "closingDate": null,
  "applicantTypes": [],
  "requirements": [],
  "requiredDocuments": [],
  "evaluationCriteria": [],
  "missingInformation": [],
  "evidence": []
}
```

Campos desconhecidos não devem ser preenchidos por inferência sem identificação.

### 7.4 RAG

O RAG deverá recuperar informações de:

- Editais.
- Anexos.
- Documentos autorizados.
- Perfil.
- Versões de projeto.
- Modelos aprovados.

Cada fragmento deverá manter:

- Documento de origem.
- Página.
- Seção.
- Hash ou versão do documento.
- Metadados de domínio.
- Identificador de organização quando aplicável.

### 7.5 Segurança da IA

- Proteção contra prompt injection.
- Separação entre instruções e dados.
- Validação de ferramentas.
- Limites de execução.
- Proteção contra vazamento cross-tenant.
- Validação de JSON.
- Tratamento de saída inválida.
- Auditoria de operações.

### 7.6 Custos

Registrar:

- Modelo.
- Tokens de entrada.
- Tokens de saída.
- Tempo.
- Operação.
- Usuário.
- Organização.
- Custo estimado.
- Status.

O sistema deve impedir consumo ilimitado.

---

## 8. MODELO DE DADOS

Banco principal: **MongoDB Atlas**.

### 8.1 Collections

```
users
organizations
memberships
professional_profiles
opportunities
opportunity_sources
opportunity_evidence
eligibility_rules
eligibility_assessments
projects
project_versions
project_opportunities
budgets
budget_items
project_activities
documents
document_versions
ai_jobs
ai_interactions
subscriptions
plans
usage_records
notifications
audit_logs
```

### 8.2 Identificadores

Utilizar ObjectId ou identificadores estáveis equivalentes.

Todas as relações devem ser explicitamente validadas.

### 8.3 Multi-tenancy

Toda consulta que envolva recursos privados deverá incluir contexto de autorização.

Não confiar exclusivamente em `organizationId` recebido do cliente.

### 8.4 Índices

Criar índices para:

- `users.email` (unique).
- `memberships.organizationId + userId`.
- `opportunities.domain + status`.
- `opportunities.closingDate`.
- `opportunities.geographicScope`.
- `projects.ownerId + status`.
- `projects.organizationId + status`.
- `documents.ownerId`.
- `subscriptions.providerSubscriptionId`.
- `notifications.userId + readAt`.

Índices compostos devem ser definidos após avaliação das consultas reais.

### 8.5 Versionamento

Conteúdo versionado deve ser preservado de forma imutável.

Atualizações deverão criar novas versões quando a natureza do dado exigir histórico.

---

## 9. ARQUITETURA TÉCNICA

### 9.1 Monorepo

Estrutura sugerida:

```
projeta/
apps/
  web/
  api/
  worker/
packages/
  contracts/
  shared/
  ai/
  config/
docs/
  business/
  product/
  architecture/
  specifications/
  decisions/
  validation/
infrastructure/
  docker/
  deployment/
  scripts/
```

### 9.2 Frontend

Next.js + TypeScript.

Responsabilidades:

- Interface.
- Navegação.
- Formulários.
- Dashboard.
- Gestão de projetos.
- Consumo da API.
- Controle visual de permissões.

### 9.3 Backend

NestJS.

Módulos:

```
IdentityModule
UsersModule
OrganizationsModule
MembershipsModule
ProfilesModule
OpportunitiesModule
OpportunitySourcesModule
EligibilityModule
ProjectsModule
ProjectVersionsModule
DocumentsModule
BudgetsModule
ActivitiesModule
AiModule
SubscriptionsModule
NotificationsModule
AuditModule
AdminModule
```

### 9.4 Workers

BullMQ + Redis.

Filas:

- opportunity-discovery
- opportunity-verification
- document-processing
- pdf-extraction
- ocr-processing
- edital-analysis
- ai-generation
- document-export
- notifications
- billing-webhooks

Workers devem possuir retry controlado, idempotência, timeout e dead-letter handling.

### 9.5 Storage

Object Storage compatível com S3.

Arquivos privados.

Acesso temporário por URLs assinadas.

### 9.6 Busca

MongoDB Atlas Search inicialmente.

Não introduzir Elasticsearch ou outro mecanismo sem necessidade comprovada.

### 9.7 Observabilidade

- Logs estruturados.
- Métricas.
- Traces.
- Identificador de correlação.
- Monitoramento de filas.
- Monitoramento de erros.
- Alertas operacionais.

---

## 10. API — CONTRATOS INICIAIS

Prefixo: `/api/v1`

### Authentication

```
POST /auth/register
POST /auth/login
POST /auth/logout
POST /auth/refresh
POST /auth/forgot-password
POST /auth/reset-password
```

### Profiles

```
GET  /profiles/me
PATCH /profiles/me
GET  /profiles/me/documents
POST /profiles/me/documents
```

### Organizations

```
POST  /organizations
GET   /organizations
GET   /organizations/{organizationId}
PATCH /organizations/{organizationId}
POST  /organizations/{organizationId}/members
PATCH /organizations/{organizationId}/members/{userId}
```

### Opportunities

```
GET    /opportunities
GET    /opportunities/{opportunityId}
GET    /opportunities/search
POST   /opportunities/{opportunityId}/favorite
DELETE /opportunities/{opportunityId}/favorite
```

### Edital analysis

```
POST  /edital-analyses
GET   /edital-analyses/{analysisId}
PATCH /edital-analyses/{analysisId}
POST  /edital-analyses/{analysisId}/reprocess
```

### Eligibility

```
POST /eligibility/assessments
GET  /eligibility/assessments/{assessmentId}
```

### Projects

```
POST  /projects
GET   /projects
GET   /projects/{projectId}
PATCH /projects/{projectId}
DELETE /projects/{projectId}
POST  /projects/{projectId}/generate-section
POST  /projects/{projectId}/review
POST  /projects/{projectId}/versions
GET   /projects/{projectId}/versions
```

### Budgets

```
POST  /projects/{projectId}/budgets
GET   /projects/{projectId}/budgets
PATCH /budgets/{budgetId}
POST  /budgets/{budgetId}/validate
GET   /budgets/{budgetId}/export
```

### Documents

```
POST   /documents
GET    /documents/{documentId}
DELETE /documents/{documentId}
GET    /documents/{documentId}/download
```

### Billing

```
GET  /billing/plans
GET  /billing/subscription
POST /billing/checkout
POST /billing/cancel
POST /billing/webhooks/{provider}
```

### Dashboard

```
GET /dashboard/summary
GET /dashboard/deadlines
GET /dashboard/recent-activity
```

Todos os endpoints devem possuir DTOs, validação, autorização e documentação OpenAPI.

---

## 11. SEGURANÇA, PRIVACIDADE E LGPD

Requisitos obrigatórios:

- Senhas com hash seguro.
- TLS.
- Proteção contra brute force.
- Rate limiting.
- Validação de entrada.
- Proteção contra NoSQL injection.
- CORS restritivo.
- Helmet.
- Controle de acesso.
- Proteção de arquivos.
- Auditoria.
- Backups.
- Gestão de consentimento.
- Política de retenção.
- Exclusão de dados conforme obrigações legais.
- Gestão de incidentes.

Documentos pessoais e institucionais não devem ser disponibilizados publicamente.

Dados de saúde que possam constituir dados pessoais sensíveis exigirão tratamento específico conforme LGPD. O módulo HEALTH deverá passar por análise jurídica e de privacidade própria antes do lançamento.

---

## 12. REQUISITOS NÃO FUNCIONAIS

- **RNF-001:** arquitetura modular.
- **RNF-002:** TypeScript strict.
- **RNF-003:** API versionada.
- **RNF-004:** testes unitários.
- **RNF-005:** testes de integração.
- **RNF-006:** testes E2E.
- **RNF-007:** autorização no backend.
- **RNF-008:** isolamento multi-tenant.
- **RNF-009:** processamento assíncrono.
- **RNF-010:** logs estruturados.
- **RNF-011:** observabilidade.
- **RNF-012:** backup automatizado.
- **RNF-013:** recuperação documentada.
- **RNF-014:** acessibilidade de interface.
- **RNF-015:** responsividade.
- **RNF-016:** internacionalização preparada, inicialmente pt-BR.
- **RNF-017:** documentação OpenAPI.
- **RNF-018:** migrations ou scripts de evolução de dados versionados.
- **RNF-019:** operações críticas idempotentes.
- **RNF-020:** arquivos com acesso controlado.
- **RNF-021:** controle de custo da IA.
- **RNF-022:** tratamento de falhas de serviços externos.

Metas iniciais de desempenho, a validar em ambiente de homologação:

- API: p95 inferior a 500 ms em consultas simples.
- Busca: p95 inferior a 1 segundo em consultas comuns.
- Jobs pesados: processamento assíncrono.
- Disponibilidade inicial pretendida: 99,5% mensal.

Essas metas não se aplicam ao tempo de inferência de modelos externos.

---

## 13. UX E JORNADAS

### Jornada J01 — Primeiro acesso

1. Acessar plataforma.
2. Criar conta.
3. Confirmar e-mail.
4. Selecionar área de atuação.
5. Criar perfil.
6. Acessar dashboard.
7. Consultar oportunidades.

### Jornada J02 — Encontrar edital

1. Abrir Radar.
2. Pesquisar oportunidades.
3. Aplicar filtros.
4. Consultar detalhes.
5. Verificar fonte.
6. Salvar oportunidade.

### Jornada J03 — Analisar edital

1. Abrir analisador.
2. Enviar PDF.
3. Acompanhar processamento.
4. Consultar análise.
5. Conferir requisitos.
6. Revisar informações.
7. Executar avaliação de elegibilidade.

### Jornada J04 — Criar projeto

1. Criar projeto.
2. Descrever ideia.
3. Selecionar domínio CULTURE.
4. Informar dados complementares.
5. Selecionar oportunidade.
6. Desenvolver seções.
7. Revisar.
8. Salvar versão.

### Jornada J05 — Preparar submissão

1. Abrir projeto.
2. Conferir edital.
3. Validar requisitos.
4. Conferir orçamento.
5. Conferir cronograma.
6. Verificar documentos.
7. Gerar DOCX/PDF.
8. Marcar proposta como pronta.
9. Registrar submissão realizada pelo usuário.

---

## 14. DESIGN SYSTEM E INTERFACE

Diretrizes:

- Interface profissional.
- Visual limpo.
- Navegação previsível.
- Responsividade.
- Hierarquia visual clara.
- Formulários com salvamento controlado.
- Feedback de processamento.
- Estados vazios úteis.
- Erros explicativos.
- Acessibilidade.

Menu principal:

- Dashboard
- Oportunidades
- Meus projetos
- Assistente IA
- Documentos
- Calendário
- Perfil
- Assinatura
- Configurações

Área administrativa separada.

Não implementar interface complexa antes de estabilizar os fluxos de produto.

---

## 15. FONTES E CONECTORES DE OPORTUNIDADES

O sistema deve possuir uma camada independente de conectores.

Interface conceitual:

```
OpportunitySourceAdapter
```

Métodos:

- `discover()`
- `fetchDetails()`
- `normalize()`
- `verify()`
- `healthCheck()`

Cada fonte deverá possuir:

- Identificação.
- URL oficial.
- Método de coleta.
- Frequência.
- Última execução.
- Status.
- Política de acesso.
- Limitações.
- Histórico de falhas.

Fontes prioritárias para pesquisa e integração:

**CULTURE:**
- Ministério da Cultura.
- SALIC.
- PNAB.
- Transferegov.
- Secretarias estaduais.
- Secretarias municipais.
- Fundações culturais.
- Institutos e patrocinadores.

Não presumir que todos esses portais oferecem APIs públicas.

Antes de implementar cada conector, verificar:

- Existência de API.
- Termos de uso.
- Política de acesso.
- Disponibilidade de dados.
- Restrições de automação.
- Método permitido de atualização.

No MVP, a administração manual assistida será aceita como mecanismo de curadoria.

---

## 16. MÉTRICAS DE PRODUTO

**Aquisição**
- Visitantes.
- Cadastros.
- Conversão de landing page.
- Custo por aquisição.

**Ativação**
- Perfis concluídos.
- Primeira oportunidade salva.
- Primeiro edital analisado.
- Primeiro projeto criado.

**Engajamento**
- Projetos ativos.
- Análises realizadas.
- Oportunidades consultadas.
- Documentos gerados.
- Usuários ativos mensais.

**Receita**
- MRR.
- ARR.
- ARPU.
- CAC.
- LTV.
- Churn.
- Conversão Free → Paid.

**Qualidade**
- Precisão da extração.
- Taxa de informações corrigidas.
- Oportunidades desatualizadas.
- Falhas de processamento.
- Custo médio por análise.
- Tempo médio de geração.

**Métrica principal (North Star Metric):**

Número de projetos ativos que avançam de uma etapa de desenvolvimento para uma etapa posterior por mês.

Não utilizar aprovação de editais como métrica isolada de sucesso, pois depende de fatores externos ao produto.

---

## 17. ROADMAP

**FASE 0 — Fundação**
- Repositório.
- Documentação.
- Arquitetura.
- Configuração.
- CI.
- Testes.
- Identidade.
- Banco de dados.

**FASE 1 — MVP Cultural**
- Cadastro.
- Perfil.
- Radar.
- Oportunidades.
- Analisador.
- Elegibilidade.
- Assistente.
- Projetos.
- Versionamento.
- Exportação.
- Billing básico.

**FASE 2 — Produtividade**
- Orçamentos.
- Cronogramas.
- Documentos.
- Alertas.
- Dashboard avançado.
- Gestão de organizações.

**FASE 3 — Domínio Saúde**
- Fontes próprias.
- Regras de elegibilidade.
- Modelos.
- Workflows.
- Validações.
- Políticas de dados.

**FASE 4 — Expansão**
- Marketplace.
- Colaboração avançada.
- Integrações institucionais.
- Relatórios.
- Analytics.

Não iniciar uma fase sem critérios de aceite da fase anterior.

---

## 18. CRITÉRIOS DE ACEITE GERAIS

Uma funcionalidade só será considerada concluída quando:

1. Requisito estiver implementado.
2. Testes correspondentes estiverem aprovados.
3. Validação de autorização estiver implementada.
4. Casos de erro estiverem tratados.
5. Documentação estiver atualizada.
6. Logs e observabilidade estiverem adequados.
7. Não houver regressões conhecidas.
8. Pull request ou revisão equivalente estiver concluída.
9. Critérios de aceite estiverem demonstrados.

Não aceitar implementação baseada apenas em screenshots ou demonstração visual.

---

## 19. ESTRATÉGIA DE TESTES

**Unitários**
- Regras de negócio.
- Cálculos financeiros.
- Elegibilidade.
- Transições de estado.
- Limites de assinatura.
- Normalização de oportunidades.

**Integração**
- MongoDB.
- Redis.
- Storage.
- Provedor de IA.
- Billing.
- Webhooks.

**E2E**
- Cadastro.
- Login.
- Perfil.
- Radar.
- Análise de edital.
- Criação de projeto.
- Versionamento.
- Exportação.
- Assinatura.

**Segurança**
- Acesso cross-tenant.
- IDOR.
- Upload malicioso.
- Injeção.
- Abuso de endpoints.
- Exposição de arquivos.
- Repetição de webhooks.

---

## 20. CRITÉRIOS DE LANÇAMENTO

O MVP poderá ser disponibilizado quando:

- Cadastro e login funcionarem.
- Usuários puderem criar perfil.
- Radar possuir oportunidades verificáveis.
- Editais puderem ser analisados.
- Compatibilidade puder ser explicada.
- Projetos puderem ser criados e editados.
- Versionamento funcionar.
- Documentos puderem ser exportados.
- Billing funcionar em ambiente de produção.
- Limites dos planos forem aplicados.
- Logs e backups estiverem operacionais.
- Testes críticos passarem.
- Política de privacidade e termos estiverem publicados.

---

## 21. DECISÕES ARQUITETURAIS INICIAIS

- **ADR-001: Monólito modular NestJS.**
  Motivo: reduzir complexidade operacional e preservar separação de domínios.
- **ADR-002: Next.js como frontend.**
  Motivo: experiência web, responsividade e produtividade.
- **ADR-003: MongoDB Atlas como banco inicial.**
  Motivo: flexibilidade documental e integração com o ecossistema atual.
- **ADR-004: Redis + BullMQ.**
  Motivo: processamento assíncrono.
- **ADR-005: LLM desacoplado.**
  Motivo: evitar dependência de um único provedor.
- **ADR-006: Cultura como domínio inicial.**
  Motivo: validar o mercado com menor escopo funcional.
- **ADR-007: Saúde como domínio futuro isolado.**
  Motivo: requisitos específicos e diferentes regras institucionais.
- **ADR-008: Fontes oficiais como autoridade.**
  Motivo: confiabilidade e rastreabilidade.
- **ADR-009: Multi-tenancy desde a fundação.**
  Motivo: permitir crescimento para consultorias e instituições.
- **ADR-010: SDD como metodologia obrigatória.**
  Motivo: preservar contexto, decisões e integridade do produto durante desenvolvimento assistido por IA.

---

## 22. DECISÕES AINDA PENDENTES

Não inventar valores ou fornecedores definitivos para:

- Provedor principal de IA.
- Provedor de pagamentos.
- Storage definitivo.
- Serviço de e-mail.
- Serviço de OCR.
- Limites finais dos planos.
- Política definitiva de retenção.
- Fontes que terão integração automatizada.
- Política comercial de testes gratuitos.
- Prazo de retenção de documentos.

Esses pontos devem ser avaliados em ADRs específicos.

---

## 23. DEFINIÇÃO DE PRONTO — DEFINITION OF DONE

Uma entrega está concluída quando:

- Requisito associado.
- Spec aprovada.
- Código implementado.
- Testes aprovados.
- Segurança validada.
- Documentação atualizada.
- Contratos atualizados.
- Sem alteração não autorizada em módulos existentes.
- Revisão concluída.
- Evidências de execução registradas.

---

## 24. DIRETRIZ FINAL

PROJETA deve ser desenvolvido como um produto comercial sustentável, não como uma demonstração de capacidades de IA.

Prioridades:

1. Resolver um problema real.
2. Garantir confiabilidade das oportunidades.
3. Facilitar o desenvolvimento de projetos.
4. Organizar documentos e processos.
5. Entregar valor recorrente.
6. Controlar custos.
7. Preservar arquitetura evolutiva.

A primeira entrega técnica é a **FASE 0 — Fundação**.

**Nenhum agente deve iniciar a implementação integral do sistema diretamente a partir deste documento.**

Primeiro deve analisar o workspace, identificar os artefatos existentes, elaborar o plano de execução, detalhar as especificações e registrar decisões arquiteturais.

A implementação deverá ocorrer em incrementos pequenos, testáveis e revisáveis.

**Fim do PRD 1.0.**
