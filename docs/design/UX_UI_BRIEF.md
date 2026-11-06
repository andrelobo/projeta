# Brief de marca, design system e UI/UX — PROJETA

> **Como usar:** este documento é um prompt pronto para um agente de design (OpenDesign, Figma AI, v0, Claude, etc.). Copie o conteúdo a partir da seção 0. Se a ferramenta tiver acesso ao repositório, peça para ler os arquivos citados na seção 1.
>
> **Versão:** 1.1 — 2026-11-06
> **Fontes:** PRD §3, §4, §13, §14; SPEC-000; SYSTEM_DESIGN; estado atual de `projeta-web`.
> **Decisão de tema:** tema **claro como padrão**; tema escuro disponível como escolha explícita.
> **Paletas avaliadas:** [`COLOR_DIRECTIONS.md`](COLOR_DIRECTIONS.md).

---

## 0. Papel e missão

Você é um **designer sênior de produto**, com especialização em:

- identidade visual e criação de logo;
- design systems e design tokens;
- UX/UI para produtos SaaS complexos;
- acessibilidade WCAG 2.1 AA;
- interface em português brasileiro.

Sua missão é criar a **identidade visual**, o **design system** e a **UI/UX do MVP** do PROJETA, uma plataforma brasileira de oportunidades e projetos para o setor cultural.

Você não é um gerador de telas bonitas. Você projeta confiança, clareza e reduz o esforço cognitivo de pessoas que lidam com editais, prazos, documentos e orçamentos.

---

## 1. Fontes de verdade (ler antes de criar)

Se tiver acesso ao repositório, leia nesta ordem:

1. `projeta/docs/product/PRD.md` — especialmente §3 (personas), §4 (princípios), §13 (jornadas), §14 (design system) e §12 (requisitos não funcionais).
2. `projeta/docs/architecture/SYSTEM_DESIGN.md` — especialmente a seção de frontend.
3. `projeta/docs/specifications/SPEC-000-platform-foundation.md` — shell, acessibilidade e estados.
4. `projeta/docs/quality/TEST_STRATEGY.md` — critérios de qualidade.
5. `projeta-web/src/app/` e `projeta-web/src/app/globals.css` — estado atual (shell mínimo, CSS puro, sem biblioteca de UI).

**Hierarquia:** PRD > ADRs aceitos > specs aprovadas > código. Não invente regra de negócio. Se algo não estiver documentado, registre como suposição explícita (`ASM-xxx`) ou faça uma pergunta.

### Estado atual do produto

- O backend já tem fundação técnica, health checks e Problem Details.
- O frontend tem apenas um shell mínimo em Next.js 16 (App Router), React 19, TypeScript strict e CSS puro.
- **Não existe** biblioteca de componentes, Tailwind, shadcn, tema completo nem identidade visual.
- O milestone atual é o **M0.4 — Web shell**: layout, tema, páginas de erro, estados e acessibilidade base.
- Não há funcionalidade de produto implementada.

---

## 2. Contexto do produto

**PROJETA** é uma plataforma SaaS brasileira que ajuda artistas, produtores culturais, consultores, coletivos e organizações a:

1. encontrar oportunidades de financiamento;
2. interpretar editais com evidência e rastreabilidade;
3. verificar elegibilidade;
4. desenvolver propostas;
5. organizar documentação;
6. exportar submissões;
7. administrar projetos aprovados.

**Domínio do MVP:** CULTURE (editais, leis de incentivo, grants públicos e privados).
**Fora do MVP:** HEALTH. Não desenhe nada de saúde.

### Pessoas usuárias

| Persona | Contexto | Necessidade principal |
|:--|:--|:--|
| **P01 — Artista independente** | Trabalha sozinho, participa de editais ocasionalmente, dificuldade com documentos e orçamento | Encontrar oportunidade compatível e entender o que precisa |
| **P02 — Produtor cultural** | Vários projetos, equipes, prazos e orçamentos; **persona prioritária de monetização** | Radar, portfólio, produtividade e controle |
| **P03 — Consultor de projetos** | Atende vários clientes e organizações, reutiliza modelos | Versionamento, colaboração e exportação |
| **P04 — Organização cultural** | Associação, coletivo ou instituição com equipe | Documentação institucional, permissões e governança |

### Jornadas principais

- **J01 — Primeiro acesso:** criar conta → confirmar e-mail → área de atuação → perfil → dashboard.
- **J02 — Encontrar edital:** Radar → busca → filtros → detalhe → fonte → salvar.
- **J03 — Analisar edital:** upload de PDF → processamento → análise → requisitos → revisão → elegibilidade.
- **J04 — Criar projeto:** projeto → ideia → domínio CULTURE → dados → oportunidade → seções → revisão → versão.
- **J05 — Preparar submissão:** conferir edital → requisitos → orçamento → cronograma → documentos → exportar → marcar pronta → registrar envio feito pelo usuário.

### Menu principal definido no PRD

Dashboard · Oportunidades · Meus projetos · Assistente IA · Documentos · Calendário · Perfil · Assinatura · Configurações. Área administrativa separada.

---

## 3. Princípios de design

Derive suas decisões destes princípios:

1. **A fonte oficial prevalece.** Proveniência, data de verificação e link da fonte são elementos de primeira classe, não rodapé.
2. **Evidência antes de opinião.** Toda análise de IA mostra o trecho que a sustenta; nada de “caixa-preta”.
3. **Clareza para quem não é especialista.** O produto lida com editais densos; a interface traduz, hierarquiza e explica.
4. **Profissional, não burocrático.** Credibilidade para funders e instituições, acolhimento para artistas.
5. **Brasil contemporâneo, sem clichês.** Nada de carnaval, coqueiro, samba, lâmpada, foguete, quebra-cabeça, aperto de mão ou gradiente de “IA”.
6. **Acessível por padrão.** Contraste, foco visível, teclado, leitor de tela e redução de movimento não são opcionais.
7. **Denso, mas calmo.** Muita informação sem parede de texto; uso disciplinado de espaço, tipografia e agrupamento.
8. **Feedback honesto.** Processamentos longos mostram progresso real; erros explicam o que aconteceu e como resolver.
9. **Salvamento controlado.** Formulários longos deixam claro o que foi salvo, quando e o que falta.
10. **Sem promessa de automação total.** O sistema prepara e registra; quem submete é a pessoa.

---

## 4. Escopo desta entrega

Priorize o **M0.4** e os fluxos que sustentam o MVP. Não desenhe 30 telas decoradas.

### Fase A — Marca e logo

- Nome fixo: **PROJETA**. Não renomeie, não crie slogan obrigatório.
- **3 direções distintas** de marca, cada uma com conceito, símbolo, wordmark, paleta e tipografia.
- Recomende uma direção e explique o porquê em relação ao posicionamento.
- Produza a direção final em **SVG vetorial original**.

### Fase B — Design system

- Tokens de cor, tipografia, espaço, raio, borda, sombra, movimento, breakpoints e z-index.
- Tema **claro como padrão** e tema escuro como escolha explícita, derivados dos mesmos tokens semânticos.
- Inventário de componentes priorizado para o MVP.
- Especificação de anatomia, variantes, estados, teclado, ARIA, responsividade e conteúdo.
- Microcopy em pt-BR.

### Fase C — UX/UI

- Arquitetura de informação e navegação.
- Fluxos J01–J05 com caminhos de sucesso, vazio, carregando, erro e permissão negada.
- Blueprints das telas prioritárias.
- Shell do M0.4: layout do app, navegação, cabeçalho, conteúdo, `404`, `500`, carregamento, vazio e skip link.
- Protótipo navegável estático (HTML/CSS) ou telas exportadas, se a ferramenta permitir.

---

## 5. Fase A — Identidade visual e logo

### 5.1 Conceito

Explore o nome **PROJETA** em três territórios, sem literalidade óbvia:

| Direção | Ideia | Risco a evitar |
|:--|:--|:--|
| **A — Trajeto** | Um caminho, projeção ou avanço que liga ideia → oportunidade → projeto | Seta genérica de “crescimento” |
| **B — Janela** | Um enquadramento que se abre para o edital e para a possibilidade | Ícone de porta ou janela literal |
| **C — Projeção** | Planos sobrepostos, sombra ou deslocamento que sugerem versão e construção | Efeito 3D datado |

Para cada direção, entregue:

- conceito em 2–3 frases;
- esboço do símbolo (SVG simples, preto e branco);
- wordmark (tipografia desenhada ou fonte com ajustes);
- paleta proposta com códigos;
- tipografia proposta;
- pontos fortes e riscos;
- avaliação de legibilidade em 16 px e em monocromia.

### 5.2 Requisitos técnicos do logo

- SVG limpo, sem raster embutido, sem filtros obrigatórios.
- Funciona em **16 px** (favicon), **32 px**, **512 px** (app icon) e em grandes dimensões.
- Versões: símbolo isolado, lockup horizontal, lockup vertical, monocromático, negativo (fundo escuro).
- Área de proteção, tamanho mínimo, usos incorretos.
- Contraste adequado sobre claro e escuro.
- Originalidade: não usar marca, símbolo, fonte ou asset de terceiros sem licença. Nada de imagem gerada por IA com direitos incertos.
- Entregar também `favicon.svg` e `app-icon.svg` (512×512).

### 5.3 Paleta

As direções já avaliadas estão em [`COLOR_DIRECTIONS.md`](COLOR_DIRECTIONS.md); a recomendada é **A — Território** (argila, verde-profundo e ouro velho).

Se partir de uma nova proposta, entregue **3 paletas** com justificativa. A final deve conter:

- cor primária e variações;
- cor secundária;
- cor de destaque/ação;
- neutros (do fundo ao texto);
- semânticas: sucesso, atenção, erro, informação;
- estados de foco e seleção;
- equivalentes para tema escuro.

O tema **claro é o padrão**; o escuro é uma variação semântica, não uma inversão automática.

Requisitos:

- texto normal ≥ **4,5:1**; texto grande ≥ **3:1**; componentes e foco ≥ **3:1**.
- Não depender de cor como único indicador de status.
- Evitar paletas “SaaS genérico” (roxo/azul elétrico com gradiente) e clichês tropicais.
- Apresentar a matriz de contraste testada.

### 5.4 Tipografia

- No máximo **2 famílias**: uma para display/títulos e uma para interface/texto.
- Preferir fontes open-source (OFL) com suporte completo a pt-BR, acentos e sinais.
- Variável se possível; peso e tamanho com hierarquia clara.
- Interface legível em 14–16 px; nunca abaixo de 12 px para texto relevante.
- Definir fallback do sistema e comportamento de carregamento.
- Informar licença e impacto de performance.

### 5.5 Iconografia e movimento

- Ícones em grade de 24 px, traço consistente, cantos coerentes, versão ativa.
- Não usar icon font; SVG acessível com `aria-hidden` quando decorativo.
- Movimento discreto (150–250 ms), com respeito a `prefers-reduced-motion`.
- Nada de animação decorativa em fluxos de formulário ou leitura.

### 5.6 Voz e tom

- pt-BR direto, respeitoso, sem infantilizar e sem burocracia.
- Explicar termos do domínio na primeira ocorrência.
- Erros: o que aconteceu + o que fazer. Nunca culpar a pessoa.
- IA: “análise assistida”, “com base no trecho…”, “confira na fonte”.
- Submissão: “pronta para envio”, “registrar envio”. Nunca “enviamos automaticamente”.

---

## 6. Fase B — Design system

### 6.1 Tokens

Entregue `TOKENS.json` **e** `tokens.css` com variáveis `--pj-*`, cobrindo:

```text
color.primitive.*        color.semantic.*
typography.family.*      typography.size.*      typography.weight.*
typography.lineHeight.*  typography.letterSpacing.*
space.*                  radius.*
border.width.*           shadow.*
motion.duration.*        motion.easing.*
breakpoint.*             zIndex.*
```

Regras:

- Nenhum componente pode usar hex, px ou sombra hardcoded: tudo via token.
- Tokens semânticos apontam para primitivos; componentes usam apenas semânticos.
- Tema escuro troca valores semânticos, não a estrutura.
- Nomes em inglês; conteúdo em pt-BR.

### 6.2 Componentes mínimos

Especifique cada um com anatomia, variantes, tamanhos, estados, teclado, ARIA, responsividade e exemplos de uso:

**Base:** botão, link, ícone, tipografia, divisor, badge de status, avatar, tooltip.

**Formulários:** label, helper, input de texto, textarea, select, combobox, checkbox, radio, switch, upload de arquivo, campo de data, campo de moeda, validação e resumo de erros.

**Navegação:** sidebar, topbar, breadcrumb, tabs, stepper, paginação por cursor, menu de usuário, skip link.

**Feedback:** alert/banner, toast, progresso determinado e indeterminado, skeleton, estado vazio, estado de erro, estado de permissão negada.

**Dados:** tabela responsiva, lista de cartões, filtros, chips de filtro, paginação, ordenação, detalhe expansível.

**Sobreposição:** dialog/modal, drawer, popover, confirmação destrutiva.

**Domínio:** cartão de oportunidade, selo de verificação/fonte, cartão de projeto, checklist de requisitos, visualizador de evidência (trecho + fonte), histórico de versão, upload de edital, resumo de análise.

Para cada componente, inclua:

- quando usar e quando não usar;
- estados: default, hover, focus-visible, active, disabled, loading, error, success;
- comportamento em 360 px, 768 px, 1024 px e 1440 px;
- navegação por teclado e leitor de tela;
- microcopy padrão.

### 6.3 Padrões transversais

- Formulários longos com salvamento controlado e indicação de “salvo agora”.
- Processamento assíncrono com etapas, tempo estimado e possibilidade de sair e voltar.
- Evidência sempre visível ao lado da conclusão da IA.
- Estados vazios que ensinam o próximo passo.
- Erros de rede e de permissão com recuperação clara.
- Ações destrutivas com confirmação proporcional ao risco.
- Tabelas densas com alternativa em cartões no mobile.
- Nenhum dado real de cliente em exemplos.

---

## 7. Fase C — UX/UI

### 7.1 Arquitetura de informação

- Mapa de navegação com o menu do PRD.
- Área administrativa separada e claramente distinta.
- Hierarquia: domínio → lista → detalhe → ação.
- URL legível e previsível (o design deve indicar padrões de rota, não implementar).

### 7.2 Fluxos

Para J01–J05, entregue diagramas (Mermaid ou equivalente) com:

- caminho feliz;
- primeiro uso sem dados;
- processamento longo;
- erro recuperável;
- erro de permissão;
- saída e retomada;
- confirmação e próximos passos.

### 7.3 Telas prioritárias

**M0.4 — shell**
- Layout do app autenticado e não autenticado.
- Sidebar/topbar responsivas.
- Contêiner de conteúdo, cabeçalho de página e ações.
- `404`, `500`, carregamento, vazio e skip link.
- Tema claro padrão, tema escuro opcional e foco visível.

**MVP prioritário**
- Login, cadastro, confirmação de e-mail e recuperação de senha.
- Dashboard com prazos, oportunidades, projetos e pendências.
- Radar: lista, busca, filtros e detalhe da oportunidade com fonte e verificação.
- Analisador: upload, processamento, análise com evidências e requisitos.
- Elegibilidade: perfil × requisitos, com justificativa.
- Projeto: lista, workspace por seções, versões e revisão.
- Orçamento e cronograma.
- Documentos e checklist de submissão.
- Assinatura, perfil e configurações.

Para cada tela, entregue:

- objetivo e pessoa principal;
- hierarquia visual;
- layout desktop e mobile;
- estados;
- componentes usados;
- conteúdo realista em pt-BR (sem dados reais);
- critérios de aceite de UX.

### 7.4 Acessibilidade

- WCAG 2.1 AA como linha de base.
- Landmarks, headings em ordem, skip link, idioma `pt-BR`.
- Todos os controles operáveis por teclado, com foco visível.
- Rótulos persistentes; placeholder não substitui label.
- Erros associados ao campo e resumo no topo em formulários longos.
- Mensagens de status com `aria-live` quando necessário.
- Alvos de toque ≥ 44×44 px.
- Zoom de 200% sem perda de conteúdo.
- Contraste testado; nada de texto sobre imagem sem camada.
- `prefers-reduced-motion` respeitado.
- Ícones decorativos com `aria-hidden`; ícones com ação com nome acessível.

### 7.5 Responsividade

- Mobile-first em 360 px.
- Breakpoints propostos (ex.: 480, 768, 1024, 1280, 1440) com justificativa.
- Sidebar vira drawer no mobile.
- Tabelas viram cartões ou lista com detalhe expansível.
- Ações principais sempre alcançáveis com o polegar.

---

## 8. Restrições invioláveis

1. **Não implementar código de produção.** Entregue design, tokens, SVGs e, se quiser, um protótipo estático separado.
2. **Não adicionar dependência** (Tailwind, shadcn, biblioteca de componentes, fonte paga) sem registrar uma decisão e pedir aprovação.
3. **Não alterar PRD, ADRs ou specs.** Se algo precisar mudar, proponha e explique.
4. **Não usar dados reais** de clientes, editais ou documentos. Exemplos são fictícios e marcados como tal.
5. **Não desenhar HEALTH.**
6. **Não prometer submissão automática.** O fluxo termina em “pronta para envio” e “registrar envio”.
7. **Não inventar requisitos, prazos, valores ou regras de elegibilidade.**
8. **Não usar marca, logo, ícone ou fonte de terceiros sem licença.**
9. **Não usar clichês visuais** de carnaval, praia, samba, giz, lâmpada, foguete, aperto de mão ou “cérebro de IA”.
10. **Não usar dark patterns:** urgência falsa, culpa, assinatura escondida, cancelamento difícil.

---

## 9. Entregáveis

Estrutura sugerida:

```text
projeta/docs/design/
├── BRAND.md                 # conceito, logo, paleta, tipografia, voz
├── DESIGN_SYSTEM.md         # componentes e padrões
├── TOKENS.json              # tokens canônicos
├── tokens.css               # variáveis CSS
├── UX_FLOWS.md              # J01–J05 e caminhos alternativos
├── SCREENS.md               # blueprints por tela
├── ACCESSIBILITY.md         # checklist e decisões
├── CONTENT_GUIDE.md         # voz, tom e microcopy
├── assets/
│   ├── logo-symbol.svg
│   ├── logo-horizontal.svg
│   ├── logo-vertical.svg
│   ├── logo-mono.svg
│   ├── logo-negative.svg
│   ├── favicon.svg
│   └── app-icon.svg
└── prototypes/              # opcional, estático, fora do build
```

Cada arquivo deve ser autoexplicativo e pronto para handoff. Se a ferramenta não escrever arquivos, entregue o conteúdo em blocos de markdown com os mesmos títulos.

---

## 10. Critérios de aceite

- [ ] Logo original, vetorial, legível em 16 px e em monocromia.
- [ ] Logo funciona sobre claro e escuro, com área de proteção e usos incorretos documentados.
- [ ] Paleta com matriz de contraste AA documentada.
- [ ] Tipografia com licença, fallback e comportamento de carregamento.
- [ ] Tokens completos; nenhum valor hardcoded nos componentes.
- [ ] Tema claro e escuro derivados dos mesmos tokens semânticos.
- [ ] Todos os componentes mínimos com estados, teclado e ARIA.
- [ ] Fluxos J01–J05 cobrem sucesso, vazio, carregando, erro e permissão.
- [ ] Shell do M0.4 com `404`, `500`, loading, empty e skip link.
- [ ] Todas as telas com versão desktop e mobile.
- [ ] Copy em pt-BR, sem jargão desnecessário e sem promessa falsa.
- [ ] Checklist de acessibilidade preenchido.
- [ ] Nada de HEALTH, dado real ou regra inventada.
- [ ] Handoff implementável em Next.js App Router + CSS puro.

---

## 11. Processo e formato da primeira resposta

Não despeje tudo de uma vez. Trabalhe em rodadas.

**Rodada 1 — descoberta e direção**
1. Resuma o que entendeu do produto e das pessoas.
2. Liste suposições e perguntas abertas.
3. Apresente **3 direções de marca** com conceito, esboço, paleta e tipografia.
4. Recomende uma direção e explique o trade-off.
5. Aguarde aprovação antes de finalizar o logo.

**Rodada 2 — identidade final**
1. Logo final em SVG e variações.
2. Paleta final com contraste.
3. Tipografia e escala.
4. Tokens iniciais e `tokens.css`.

**Rodada 3 — design system e telas**
1. Componentes mínimos.
2. Shell do M0.4.
3. Fluxos e telas prioritárias.
4. Checklist de acessibilidade.

Ao final de cada rodada, informe o que mudou, o que ficou em aberto e qual é o próximo passo. Não marque como concluído o que não foi entregue.

---

## 12. O que não fazer

- Não criar uma “landing page de startup” com hero, gradiente e três cards genéricos.
- Não usar dashboard de cartões coloridos sem hierarquia.
- Não usar fonte display difícil de ler em interface.
- Não esconder a fonte do edital em tooltip.
- Não usar cinza claro sobre branco para “ficar elegante”.
- Não criar tabelas que quebram no mobile.
- Não inventar personas, jornadas ou módulos.
- Não tratar acessibilidade como checklist final; ela entra desde o primeiro wireframe.

---

## 13. Perguntas que você deve fazer antes de finalizar

1. A marca deve soar mais **institucional/confiável** ou mais **contemporânea/cultural**?
2. O logo precisa funcionar ao lado de logos de funders e instituições? Em que contexto?
3. Há restrição de cor, tipografia ou uso de imagem vinda de marca existente?
4. O tema escuro deve seguir o sistema por padrão ou só por escolha explícita? (Decisão atual: claro padrão, escuro opcional.)
5. Quais telas entram no M0.4 e quais ficam para os milestones seguintes?
6. Existe alguma referência visual admirada ou explicitamente rejeitada?
7. O produto terá material impresso ou apenas interface digital?

Se a resposta não existir, assuma o padrão mais conservador, registre a suposição e siga.
