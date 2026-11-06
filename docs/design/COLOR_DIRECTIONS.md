# Direções de cor — PROJETA

> **Status:** proposta para decisão (2026-11-06)
> **Escopo:** marca e design system do MVP. Não é implementação.
> **Relacionado:** [`UX_UI_BRIEF.md`](UX_UI_BRIEF.md), PRD §14, SPEC-000.
> **Tema claro é o padrão.** O tema escuro existe como escolha explícita e é derivado dos mesmos tokens semânticos.

Este documento avalia quatro direções cromáticas para um produto cultural, social, de saúde (extensão futura) e de ativismo — sem cair em carnaval, praia, “SaaS roxo” ou gradiente de IA.

---

## 1. Premissas

1. **Claro primeiro.** A primeira visita abre em tema claro. O tema escuro é escolha da pessoa (com opção “seguir o sistema”), nunca imposto por `prefers-color-scheme` sem consentimento.
2. **Cor não é decoração.** Cada cor tem papel: marca, ação, apoio, estado ou dado.
3. **Semântico antes de bonito.** Componentes usam tokens semânticos (`text`, `surface`, `primary`), nunca hex direto.
4. **Contraste mínimo:** 4,5:1 para texto normal, 3:1 para texto grande, componentes e foco (WCAG 2.1 AA).
5. **Status nunca depende só de cor:** sempre ícone + rótulo.
6. **A marca não muda por domínio.** CULTURE, SOCIAL e HEALTH usam o mesmo núcleo; o domínio entra como acento e conteúdo, não como rebuild visual.
7. **Sem clichês:** nada de verde-limão “sustentável”, azul-hospital, vermelho-cruz, arco-íris genérico, folha, globo, aperto de mão ou lâmpada.

---

## 2. As quatro direções

| # | Nome | Base | Acento | Personalidade | Risco |
|:--|:--|:--|:--|:--|:--|
| **A** | **Território** | argila / terracota | verde-profundo + ouro velho | quente, humano, cultural, confiável | pode ficar “artesanal demais” se o ouro dominar |
| **B** | **Sinal** | azul-noite | amarelo-sinal + vermelho-tijolo | cívico, editorial, ativista, institucional | azul pode soar “governo/genérico” |
| **C** | **Cuidado Vivo** | verde-sálvia | argila + oliva | calmo, social, saúde, acolhedor | pouco contraste de personalidade se tudo for pastel |
| **D** | **Papel & Tinta** | vermelho-tijolo | azul-petróleo + ocre | editorial, cultural, memorável | mais difícil de equilibrar em telas densas |

### Recomendação: **A — Território**

É a direção que melhor cobre o público real do MVP: artistas, produtores, consultores e organizações. A argila dá calor e identidade cultural sem folclore; o verde-profundo carrega credibilidade e diálogo com o social; o ouro entra como sinal de oportunidade e prazo, em pequena área. Também é a base que melhor se estende para HEALTH no futuro: basta promover o verde como acento de domínio, sem trocar a marca.

---

## 3. Direção A — Território (recomendada)

### 3.1 Intenção

- **Argila:** terra, território, trabalho cultural, pertencimento.
- **Verde-profundo:** construção, cuidado, sustentabilidade de projeto.
- **Ouro velho:** oportunidade, prazo, conquista — usado como **preenchimento**, nunca como texto sobre branco.
- **Papel quente:** reduz o branco clínico e aproxima de caderno, edital impresso e ateliê.

### 3.2 Tokens — tema claro (padrão)

| Token semântico | Hex | Uso |
|:--|:--|:--|
| `bg-canvas` | `#FAF8F5` | fundo geral |
| `bg-surface` | `#FFFFFF` | cartões, modais, tabelas |
| `bg-subtle` | `#F0EBE4` | cabeçalhos de tabela, hover, faixas |
| `border-subtle` | `#D9D2C7` | divisórias decorativas |
| `border-strong` | `#857C71` | borda de input, contorno de componente |
| `text` | `#1C1917` | texto principal |
| `text-muted` | `#5A534A` | texto secundário, metadados |
| `primary` | `#9E3B26` | ação principal, marca |
| `primary-hover` | `#7E2E1D` | hover/pressed |
| `primary-soft` | `#F6E3DD` | fundo de selo, item ativo suave |
| `accent` | `#1F6F63` | link, ação secundária, apoio |
| `accent-soft` | `#E3F0ED` | fundo informativo |
| `highlight` | `#C98A2E` | prazo, oportunidade, destaque — **fill** |
| `focus` | `#0B5FA5` | anel de foco (não é cor de marca) |
| `success` | `#2F6B47` | confirmação |
| `success-soft` | `#E6F2EA` | fundo de confirmação |
| `warning` | `#8A5A00` | atenção, prazo próximo |
| `warning-soft` | `#FDF0D2` | fundo de atenção |
| `danger` | `#A3241C` | erro, ação destrutiva |
| `danger-soft` | `#FBE7E5` | fundo de erro |
| `info` | `#1D5A8A` | informação |
| `info-soft` | `#E5EFF8` | fundo de informação |

### 3.3 Tokens — tema escuro (escolha explícita)

| Token semântico | Hex | Uso |
|:--|:--|:--|
| `bg-canvas` | `#16130F` | fundo geral |
| `bg-surface` | `#201A15` | cartões, modais |
| `bg-subtle` | `#2A231C` | hover, faixas |
| `border-subtle` | `#3B332A` | divisórias |
| `border-strong` | `#7A6F61` | borda de input |
| `text` | `#F5F0E8` | texto principal |
| `text-muted` | `#BDB3A6` | texto secundário |
| `primary` | `#E89B7E` | ação principal |
| `primary-hover` | `#F0B49B` | hover |
| `primary-soft` | `#3A241C` | fundo de selo |
| `accent` | `#6FC3B2` | link, apoio |
| `accent-soft` | `#17332C` | fundo informativo |
| `highlight` | `#E8B75E` | destaque — fill |
| `focus` | `#7FC7F0` | anel de foco |
| `success` | `#7FC7A0` | confirmação |
| `warning` | `#E8B75E` | atenção |
| `danger` | `#F0907E` | erro |
| `info` | `#8FBEE8` | informação |

> No escuro, **não use preto puro nem branco puro**. O papel quente vira carvão quente, para manter a mesma família perceptiva.

### 3.4 Matriz de contraste (medida, não estimada)

| Combinação | Contraste | AA |
|:--|--:|:--|
| `text` sobre `bg-canvas` (claro) | 16,3:1 | ✅ AAA |
| `text-muted` sobre `bg-canvas` | 7,15:1 | ✅ AAA |
| branco sobre `primary` | 6,76:1 | ✅ AAA |
| branco sobre `primary-hover` | 9,15:1 | ✅ AAA |
| `primary` sobre `bg-canvas` | 6,38:1 | ✅ AA |
| `accent` sobre `bg-canvas` | 5,64:1 | ✅ AA |
| `focus` sobre `bg-canvas` | 6,2:1 | ✅ AA |
| `success` sobre `bg-canvas` | 5,98:1 | ✅ AA |
| `warning` sobre `bg-canvas` | 5,59:1 | ✅ AA |
| `danger` sobre `bg-canvas` | 7,01:1 | ✅ AAA |
| `info` sobre `bg-canvas` | 6,87:1 | ✅ AAA |
| `text` sobre `highlight` | 5,96:1 | ✅ AA |
| `highlight` sobre `bg-canvas` | 2,77:1 | ❌ **não usar como texto** |
| `border-strong` sobre `bg-surface` | 4,1:1 | ✅ 3:1 (componente) |
| `text` sobre `bg-canvas` (escuro) | 16,32:1 | ✅ AAA |
| `text-muted` sobre `bg-canvas` (escuro) | 8,96:1 | ✅ AAA |
| `primary` sobre `bg-canvas` (escuro) | 8,31:1 | ✅ AAA |
| `accent` sobre `bg-canvas` (escuro) | 8,93:1 | ✅ AAA |
| `focus` sobre `bg-canvas` (escuro) | 9,99:1 | ✅ AAA |

**Regra do ouro:** `highlight` é cor de **preenchimento** (badge, barra, ponto, fundo de cartão). Texto sobre ouro é sempre `text` escuro; nunca ouro como texto sobre papel.

---

## 4. Direção B — Sinal

Para quando a marca precisar soar mais **cívica e institucional**, com energia de cartaz.

| Papel | Claro | Escuro |
|:--|:--|:--|
| Fundo | `#F7F8FA` | `#0F141B` |
| Texto | `#14171C` | `#EEF2F7` |
| Primária | `#14365C` | `#8FB6E8` |
| Secundária | `#C8452F` | `#F0907E` |
| Destaque | `#F0B429` | `#FFC94D` |
| Foco | `#0B5FA5` | `#7FC7F0` |

Contrastes-chave: branco sobre `#14365C` = **12,27:1**; texto `#14171C` sobre amarelo = **9,64:1**; `#C8452F` sobre fundo = **4,54:1** (limite AA para texto normal); `#F0B429` sobre fundo = **1,86:1** (só preenchimento).

**Quando escolher:** se a marca for usada ao lado de logotipos de fundações, governos e editais, e se o discurso de ativismo for parte central. **Custo:** aproxima do vocabulário visual de campanha política e serviço público.

---

## 5. Direção C — Cuidado Vivo

Para um posicionamento mais **social e de saúde**, sem estética hospitalar.

| Papel | Claro | Escuro |
|:--|:--|:--|
| Fundo | `#FAF9F5` | `#111713` |
| Texto | `#17201C` | `#EDF2EE` |
| Primária | `#2E6B57` | `#7FC3A8` |
| Acento | `#C2703B` | `#E0A277` |
| Apoio | `#6B7F4A` | `#B5C48C` |
| Foco | `#0B5FA5` | `#7FC7F0` |

Contrastes-chave: branco sobre `#2E6B57` = **6,26:1**; `#C2703B` sobre fundo = **3,7:1** (só texto grande/gráfico); versão para texto: `#9C4F2A` = **5,59:1**; `#6B7F4A` sobre fundo = **4,19:1** (não usar em texto pequeno).

**Quando escolher:** se o produto nascer mais próximo de saúde e políticas públicas do que de cultura. **Custo:** perde parte da energia cultural e pode ficar monótono em dashboards densos.

---

## 6. Direção D — Papel & Tinta

Editorial e memorável, com cara de publicação cultural séria.

| Papel | Claro | Escuro |
|:--|:--|:--|
| Fundo | `#FBF9F4` | `#14120F` |
| Texto | `#1B1815` | `#F4EFE7` |
| Primária | `#A9382A` | `#E58A7A` |
| Secundária | `#1F4E79` | `#86B4E0` |
| Destaque | `#D98E04` | `#E9B45C` |
| Foco | `#0B5FA5` | `#7FC7F0` |

Contrastes-chave: `#A9382A` sobre fundo = **6,07:1**; `#1F4E79` = **8,23:1**; `#D98E04` = **2,55:1** (só preenchimento); texto escuro sobre ocre = **6,58:1**.

**Quando escolher:** se a marca quiser se parecer com uma revista/plataforma editorial. **Custo:** o vermelho-tijolo compete com estados de erro e exige disciplina.

---

## 7. Como a paleta cobre os quatro nichos

A marca é uma só; o **domínio** muda o acento, o conteúdo e os ícones.

| Contexto | Papel da cor | Aplicação |
|:--|:--|:--|
| **Cultural** | argila + ouro | identidade padrão; oportunidade e prazo em ouro |
| **Social** | verde + argila | projetos coletivos, território, comunidade; evitar tom assistencialista |
| **Saúde (futuro)** | verde-sálvia/teal como acento de domínio | nada de azul-hospital, cruz vermelha ou verde-limão; HEALTH não entra no MVP |
| **Ativismo** | vermelho-tijolo + amarelo-sinal em campanhas | alto contraste, tipografia forte, sem gradiente e sem estética de partido |

> HEALTH é **extensão arquitetural**, não tela do MVP. A paleta apenas precisa não quebrar quando ele chegar.

---

## 8. Regras de uso

1. **Proporção 60/30/10:** 60% neutros de fundo, 30% superfícies/texto, 10% marca e acento.
2. **Uma ação primária por tela.** O resto é secundário ou link.
3. **Ouro nunca é texto.** É preenchimento, ícone ou barra.
4. **Status = ícone + rótulo + cor.** Daltonismo não pode custar informação.
5. **Foco sempre visível:** anel de 3 px, offset de 2 px, cor `focus`, independente do fundo.
6. **Sem gradientes de marca, sombras coloridas ou vidro fosco.** O produto é sobre clareza.
7. **Links em texto corrido são sublinhados.** Cor sozinha não basta.
8. **Tema escuro não é inversão automática:** trocar apenas tokens semânticos, manter hierarquia e contraste.
9. **Gráficos:** no máximo 6 cores categóricas, sempre com legenda, rótulo direto e padrão alternativo (traço/forma).
10. **Imagens:** se houver foto, usar camada de contraste; nunca texto solto sobre imagem.

---

## 9. Decisão pendente

- [ ] Aprovar a direção **A — Território** como base da marca.
- [ ] Confirmar que o tema claro é o padrão e o escuro é escolha explícita.
- [ ] Confirmar se o produto precisa de material impresso (muda contraste e CMYK).

Aprovada a direção, o próximo passo é gerar `tokens.css` e aplicar no shell do M0.4 — sem adicionar biblioteca de UI.
