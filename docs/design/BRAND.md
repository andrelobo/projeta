# Marca PROJETA

> **Versão:** 1.0 — 2026-11-06
> **Direção aprovada:** contemporânea cultural, com leitura institucional.
> **Base cromática:** direção A — Território (`COLOR_DIRECTIONS.md`).
> **Status:** identidade inicial para o M0.4. O logo final ainda passa por refinamento de produção antes de uso público.

---

## 1. Posicionamento

**PROJETA** organiza o caminho entre uma ideia cultural e uma proposta viável. Não é um “gerador de projetos” nem uma promessa de aprovação: é uma plataforma de trabalho, evidência e organização.

A marca precisa soar:

- **contemporânea**, não artesanal;
- **cultural**, não folclórica;
- **institucional o suficiente** para aparecer ao lado de fundações, secretarias e editais;
- **humana o suficiente** para não parecer software de banco.

### O que a marca não é

Carnaval, praia, samba, tambor, folha, globo, aperto de mão, lâmpada, foguete, quebra-cabeça, cérebro de IA, gradiente roxo, “startup de inovação”.

---

## 2. Conceito: projeção concreta

Projetar é lançar uma ideia à frente e dar forma ao que ainda não existe. O símbolo parte de um **P sólido**, com o bojo aberto, e um **quadrado verde** que ocupa o canto superior direito — a oportunidade que ainda não faz parte da letra, mas já está no horizonte.

O quadrado também dialoga com:

- **registro de impressão** e marcas de recorte da cultura gráfica;
- **janela** que se abre para o edital;
- **módulo** que se encaixa em um projeto maior.

É um símbolo de duas peças, sem efeito 3D, sem gradiente e sem metáfora óbvia. Funciona em uma cor, em 16 px e ao lado de logotipos institucionais.

---

## 3. Direções exploradas

| Direção | Conceito | Força | Risco | Decisão |
|:--|:--|:--|:--|:--|
| **A — P Aberto** | P sólido + quadrado de oportunidade | legível, contemporâneo, institucional | o quadrado pode parecer um “cursor” | **recomendada** |
| B — Plano Projetado | quadrado que projeta um paralelogramo | elegante, abstrato, escalável | não remete ao nome; genérico | descartada |
| C — Assinatura | P desenhado em traço contínuo + linha de território | cultural, autoral | frágil em 16 px e em impressão pequena | descartada |

A direção B foi testada e perde leitura de marca em contextos institucionais. A direção C foi descartada por não sobreviver ao favicon e ao carimbo.

---

## 4. Construção do símbolo

Grade de 64 × 64, sem perspectiva.

```text
P: stem 12 × 44 (x 8–20, y 12–56)
   braço superior 28 × 14 (x 20–48, y 12–26)
   lateral do bojo 14 × 10 (x 34–48, y 26–36)
   braço inferior 24 × 10 (x 20–44, y 36–46)
Quadrado: 14 × 14, raio 2 (x 44–58, y 6–20)
```

- O quadrado é desenhado **atrás** do P; a sobreposição é intencional.
- O bojo é fechado, mas o vão interno é generoso para sobreviver à redução.
- Não há stroke: todas as formas são cheias.
- O símbolo nunca é rotacionado, inclinado ou deformado.

### Área de proteção

Use `x` = altura do quadrado (14 unidades no grid de 64). Nenhum texto, borda ou outro logo entra nessa margem.

### Tamanhos mínimos

| Aplicação | Mínimo |
|:--|:--|
| Símbolo digital | 16 px |
| Símbolo impresso | 8 mm |
| Lockup horizontal digital | 96 px de largura |
| Lockup horizontal impresso | 30 mm de largura |
| Lockup vertical digital | 80 px de largura |
| Lockup vertical impresso | 25 mm de largura |

---

## 5. Variações

| Arquivo | Uso |
|:--|:--|
| `assets/logo-symbol.svg` | cor, fundos claros |
| `assets/logo-symbol-mono.svg` | uma cor, carimbo, fax, bordado |
| `assets/logo-symbol-negative.svg` | fundos escuros |
| `assets/logo-horizontal.svg` | assinatura principal |
| `assets/logo-horizontal-mono.svg` | coexistência com funders em preto |
| `assets/logo-horizontal-negative.svg` | capa escura, palco, tela |
| `assets/logo-vertical.svg` | materiais quadrados, redes sociais |
| `assets/logo-vertical-negative.svg` | fundo escuro |
| `assets/favicon.svg` | 16–32 px |
| `assets/app-icon.svg` | 512 px, ícone de aplicativo |

### Co-branding

Quando o logo aparece ao lado de realizadores, funders ou instituições:

1. use a versão **monocromática**;
2. iguale a **altura óptica** do símbolo, não a largura do lockup;
3. separe com uma linha vertical de 1 px ou espaço equivalente a `2x`;
4. nunca coloque o logo dentro de uma caixa ou moldura de terceiros.

---

## 6. Tipografia

| Papel | Fonte | Licença | Uso |
|:--|:--|:--|:--|
| Display e wordmark | **Archivo** (variável) | OFL 1.1 | títulos, números grandes, wordmark |
| Interface e texto | **IBM Plex Sans** | OFL 1.1 | corpo, formulários, tabelas, microcopy |
| Dados e código | **IBM Plex Mono** | OFL 1.1 | IDs, trechos, valores técnicos |

**Por que essa dupla:** Archivo traz uma grotesca contemporânea com presença editorial, sem o ar “startup” da Inter. IBM Plex Sans tem boa legibilidade em pt-BR, personalidade humanista e não parece interface genérica.

### Escala

| Token | Tamanho | Line-height | Uso |
|:--|--:|--:|:--|
| `text-xs` | 12 px | 1,4 | metadados, legendas |
| `text-sm` | 14 px | 1,5 | apoio, tabelas densas |
| `text-base` | 16 px | 1,6 | corpo padrão |
| `text-lg` | 18 px | 1,55 | introdução |
| `text-xl` | 20 px | 1,4 | subtítulo |
| `text-2xl` | 24 px | 1,3 | título de seção |
| `text-3xl` | 30 px | 1,2 | título de página |
| `text-4xl` | 36 px | 1,15 | destaque |
| `text-5xl` | 48 px | 1,1 | hero editorial |

- Corpo nunca abaixo de 14 px; 16 px é o padrão.
- Títulos usam Archivo 600; corpo usa IBM Plex Sans 400/500.
- **Não use caixa alta em textos longos.** Caixa alta só em labels curtos e com `letter-spacing: 0.08em`.
- Carregue as fontes localmente (`next/font/local`), em WOFF2, com `font-display: swap`. Não dependa de CDN de fontes.

---

## 7. Cor

A paleta completa, com contraste medido e tema escuro, está em [`COLOR_DIRECTIONS.md`](COLOR_DIRECTIONS.md). Resumo:

| Papel | Claro | Escuro |
|:--|:--|:--|
| Fundo | `#FAF8F5` | `#16130F` |
| Texto | `#1C1917` | `#F5F0E8` |
| Primária | `#9E3B26` | `#E89B7E` |
| Acento | `#1F6F63` | `#6FC3B2` |
| Destaque | `#C98A2E` | `#E8B75E` |
| Foco | `#0B5FA5` | `#7FC7F0` |

**Tema claro é o padrão.** O escuro é escolha explícita da pessoa, com opção “seguir o sistema”.

### Uso

- 60% neutros, 30% texto/superfície, 10% cor de marca.
- Ouro é **preenchimento**, nunca texto sobre papel.
- O verde é acento de apoio e de domínio; não compete com a ação primária.
- Não use gradientes, sombras coloridas ou transparências decorativas.

---

## 8. Impressão

O produto terá material impresso: propostas, relatórios, cartões e apresentações.

### Cores de referência (CMYK aproximado)

| Cor | Hex | CMYK de referência |
|:--|:--|:--|
| Argila | `#9E3B26` | `0 / 63 / 76 / 38` |
| Verde | `#1F6F63` | `72 / 0 / 10 / 56` |
| Ouro | `#C98A2E` | `0 / 32 / 77 / 21` |
| Papel | `#FAF8F5` | `0 / 1 / 2 / 2` |
| Tinta | `#1C1917` | `0 / 0 / 0 / 100` (texto) |

- São **aproximações para prova**. Feche a cor em prova física antes de tiragem.
- Para grandes áreas de preto, use preto rico somente em imagem; texto continua 100K.
- Prefira papel não revestido, branco quente. Evite couchê brilhante.
- O logo em uma cor usa 100K ou a cor de marca, nunca uma quarta cor.
- Em impressão de pequeno formato, respeite o mínimo de 8 mm para o símbolo.
- O símbolo não recebe relevo, hot stamping ou laminação colorida sem aprovação.

---

## 9. Voz e tom

Direto, respeitoso, sem infantilizar e sem burocracia.

| Situação | Escreva | Não escreva |
|:--|:--|:--|
| Erro de formulário | “Informe um e-mail válido para continuar.” | “Ops! Algo deu errado 😅” |
| Análise de edital | “Com base no trecho 3.2, o documento exige…” | “A IA entendeu tudo!” |
| Submissão | “Proposta pronta para envio.” | “Enviamos sua proposta automaticamente.” |
| Prazo | “Fecha em 3 dias (12/11).” | “Corra! Últimas vagas!” |
| Permissão | “Seu perfil não tem permissão para editar este orçamento.” | “Acesso negado.” |

- Explique termos do domínio na primeira ocorrência.
- Nunca culpe a pessoa pelo erro.
- Nunca prometa aprovação, elegibilidade automática ou envio automático.

---

## 10. Aplicações mínimas

- **Assinatura de e-mail:** lockup horizontal, 160 px, sobre papel.
- **Documento exportado:** capa com símbolo, título em Archivo, corpo em IBM Plex Sans, rodapé discreto.
- **Apresentação:** fundo papel, títulos em argila, dados em verde, ouro só em destaque.
- **Redes sociais:** avatar com símbolo; capa com lockup horizontal e respiro.
- **Camiseta/bolsa:** símbolo mono, uma cor, sem fundo.
- **Palco/backdrop:** lockup negative, alto contraste, sem textura.

---

## 11. Checklist de aprovação

- [ ] Símbolo legível em 16 px e em monocromia.
- [ ] Lockup horizontal funciona ao lado de logos institucionais.
- [ ] Versões negative e mono conferidas.
- [ ] Contraste AA em claro e escuro.
- [ ] Área de proteção e tamanhos mínimos documentados.
- [ ] CMYK de referência com prova física pendente.
- [ ] Fontes com licença OFL e plano de self-host.
- [ ] Nenhum clichê visual ou asset de terceiros.
