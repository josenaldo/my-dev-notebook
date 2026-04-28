---
title: "Como usar este vault"
created: 2026-04-01
updated: 2026-04-27
type: how-to
status: seedling
tags:
  - guia
  - meta
publish: false
---

# Como usar este vault

Este vault é um caderno digital de desenvolvimento (commonplace book) organizado para estudo, consulta rápida e preparação de entrevistas técnicas internacionais. O conteúdo flui em **5 zonas numeradas** que refletem o pipeline cognitivo: bruto → destilado → integrado → curatorial.

## As 5 zonas

```text
00-Meta/         meta-linguagem do codex (templates, guia, dicionário, workflow)
01-Pergaminhos/  links brutos e recursos incipientes
02-Glosas/       fichamentos de artigos lidos (uma ficha por leitura)
03-Domínios/     conhecimento integrado e evergreen, organizado por área
04-Sendas/       caminhos curatoriais que sequenciam Domínios pra estudo
```

Classificação semântica:

| Zona            | Conteúdo                                                        |
| --------------- | --------------------------------------------------------------- |
| Pergaminhos     | conteúdo bruto e links                                          |
| Glosas          | conteúdo classificado e anotado (fichamentos)                   |
| Domínios        | conteúdo organizado, agrupado e estruturado em áreas            |
| Sendas          | itens de Domínios organizados em trilha pra atender um objetivo |

## `00-Meta/` — meta-linguagem do codex

- `guia/` — como usar este vault (você está aqui)
- `templates/` — templates Templater (Nota, Glosa, How-To, TIL, MOC, Mestre, Interview Note)
- `mestres/` — referências sobre desenvolvedores e mentores
- `workflow.md` — fluxo completo (captura → destilação → integração)
- `Dicionario de Magia Tecnomante.md` — vocabulário do grimório

## `01-Pergaminhos/` — captura e curadoria inicial

- `entradas.md` — links coletados, em seções por tema (`# Tema`)
- `avaliar.md` — wishlist de tópicos a estudar
- `recursos/` — Brag Document, listas de cursos, Curso Fullcycle 3.0 (notas incipientes que serão reclassificadas conforme amadurecem)

Pergaminhos é zona de captura. Links chegam aqui, são processados (viram Glosas) e removidos. Se `entradas.md` cresce, é sinal de leitura acumulada — pause e processe.

## `02-Glosas/` — fichamentos de artigos

Cada arquivo é uma ficha de leitura: TL;DR + Pontos-chave + Citações + Meu comentário + Ver também. Filename: `<ano>-<slug>.md`. Populado pela skill `/glosa` (recomendado) ou manualmente via Templater com `Template - Glosa`.

## `03-Domínios/` — conhecimento evergreen

Notas atômicas, organizadas por área. Cada pasta tem um MOC (ex: `Java/Java.md`) que serve de índice.

- **Agnósticas**: `Fundamentos/`, `Arquitetura/`, `Ferramentas/`
- **Stacks**: `Java/`, `JavaScript/`, `Python/`, `Go/`
- **Suporte**: `Infraestrutura/`, `Inglês/`
- **Carreira**: `Entrevistas/` (Behavioral, STAR, System Design Practice, etc.)
- **Tecnologia**: `RPA/`

## `04-Sendas/` — caminhos curatoriais

Sendas são trilhas de estudo que **sequenciam Domínios** num caminho que atende a um objetivo. Não contêm conhecimento próprio — apontam pra Domínios via wikilinks.

- `Senda Entrevistas.md` — preparação para entrevistas internacionais
- `Senda Java.md`, `Senda Backend.md`, `Senda Frontend.md`, ...
- `Sendas.md` — MOC de todas as sendas

## Como navegar

1. **MOC da pasta**: cada pasta de Domínio tem um índice (`Java.md`, `Arquitetura.md`)
2. **[[Senda Entrevistas]]**: roteiro sequencial pra preparação de entrevistas
3. **[[Sendas]]**: MOC de todas as sendas
4. **[[workflow]]**: fluxo do codex e da skill `/glosa`
5. **Wikilinks** `[[Nome]]` conectam conceitos entre zonas

## Skill `/glosa` — fichamento automático

A skill `/glosa` automatiza o passo "captura → fichamento" do pipeline.

**Invocação:**

- `/glosa <url>` — slash command, forma canônica
- `Claude, fiche esse link: <url>` — linguagem natural

**O que faz (5 passos):**

1. Faz `WebFetch` da URL e extrai título, autor, site, data, idioma, body
2. Gera draft em PT-BR (TL;DR, Pontos-chave, Tags) e seleciona Citações verbatim na língua original
3. Deixa **Meu comentário** vazio com placeholder — campo onde a ficha vira sua
4. Escreve em `02-Glosas/<ano>-<slug>.md` (em colisão de slug, sufixo `-2`, `-3`)
5. Remove o link de `01-Pergaminhos/entradas.md` se ele estava lá

**Edge cases (skill avisa e aborta):**

- PDFs (URL `.pdf`)
- YouTube/Vimeo (vídeos)
- Spotify/podcasts
- Twitter/X (rede social)
- URL malformada

Detalhes completos em [[workflow]].

## Templates disponíveis

| Template                  | Quando usar                                 |
| ------------------------- | ------------------------------------------- |
| Template - Nota           | Conceitos genéricos                         |
| Template - Glosa          | Fichamento de artigo lido                   |
| Template - Interview Note | Conceitos com foco em entrevista (bilíngue) |
| Template - How-To         | Passo-a-passo de como fazer algo            |
| Template - TIL            | Algo que aprendi hoje                       |
| Template - MOC            | Novo índice de área                         |
| Template - Mestre         | Referência de desenvolvedor/mentor          |

## Como usar as seções bilíngues

Cada nota com o template **Interview Note** tem duas seções especiais:

### "Na prática (da minha experiência)"

Exemplos reais dos seus projetos. Use para praticar como contar suas experiências em entrevistas.

### "How to explain in English"

Como articular o conceito em inglês durante uma entrevista. Inclui vocabulário-chave (`termo_pt → english_term`). Leia em voz alta para praticar.

## Senda Entrevistas

A [[Senda Entrevistas]] é um roteiro sequencial com checkboxes. Siga as fases na ordem:

1. **Fundamentos** — base sólida que todo senior precisa articular
2. **Arquitetura & System Design** — o diferencial de senior
3. **Stack Java** — profundidade no backend principal
4. **Stack JavaScript/TypeScript** — fullstack
5. **Behavioral & Comunicação** — como se vender em inglês
6. **Prática** — mock interviews e exercícios

Marque `- [x]` quando completar cada item.
