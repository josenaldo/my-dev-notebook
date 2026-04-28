---
title: "Workflow do Codex"
created: 2026-04-27
updated: 2026-04-27
type: how-to
status: seedling
tags:
  - guia
  - meta
publish: false
---

# Workflow do Codex Technomanticus

Este vault é organizado em **5 zonas** que refletem o pipeline cognitivo de um commonplace book pessoal.

## As 5 zonas

```text
00-Meta/         meta-linguagem do codex (templates, guia, recursos, mestres)
01-Pergaminhos/  links brutos coletados, ainda não processados
02-Glosas/       fichamentos de artigos lidos
03-Domínios/     conhecimento integrado e evergreen, organizado por área
04-Sendas/       caminhos curatoriais que sequenciam Domínios pra estudo
```

Classificação semântica:

| Zona            | Conteúdo                                                         |
| --------------- | ---------------------------------------------------------------- |
| Pergaminhos     | conteúdo bruto e links                                           |
| Glosas          | conteúdo classificado e anotado (fichamentos)                    |
| Domínios        | conteúdo organizado, agrupado e estruturado em áreas             |
| Sendas          | itens de Domínios organizados em trilha pra atender um objetivo  |

## Como o material flui

1. **Captura.** Você acha um link interessante → cola em `01-Pergaminhos/entradas.md`, sob uma seção temática (`# Tema`)
2. **Destilação.** Lê o artigo → invoca `/glosa <url>` → ficha aparece em `02-Glosas/<ano>-<slug>.md`
3. **Limpeza automática.** A skill `/glosa` remove o link de `01-Pergaminhos/entradas.md` se ele estava lá
4. **Integração** (opcional, sob demanda). Se um artigo merece estudo profundo, você o relê e extrai notas atômicas pra `03-Domínios/<área>/`
5. **Curadoria** (opcional). Pra preparar entrevista ou estudo focado, você sequencia notas de Domínios numa Senda em `04-Sendas/Senda <Tema>.md`

## Skill `/glosa`

Invocação:

- `/glosa <url>` — slash command, forma canônica
- "Claude, fiche esse link: <url>" — linguagem natural

Comportamento:

- Faz `WebFetch` da URL e extrai título, autor, site, data, idioma, body
- Gera draft com **TL;DR**, **Pontos-chave**, **Citações** (verbatim na língua original) e **Tags**
- Deixa **Meu comentário** vazio (é onde a ficha vira sua)
- Deixa **Ver também** vazio (você preenche os wikilinks)
- Escreve o arquivo em `02-Glosas/<ano>-<slug>.md`
- Remove a linha de `01-Pergaminhos/entradas.md` se o link estava lá

Limitações do MVP:

- Só artigos web (HTML)
- PDFs, vídeos do YouTube, podcasts: não suportados — a skill avisa e aborta

## Convenções

- **Filename de Glosas**: `<ano>-<slug-kebab-case>.md` (ex: `2026-ai-now-writes-97-of-my-code.md`). Em colisão de slug no mesmo ano: sufixo `-2`, `-3`, etc.
- **`publish: false`** em Glosas (não vão pro site público via Quartz)
- **Sendas**: 1 arquivo por senda, no formato `Senda <Tema>.md` em `04-Sendas/` (flat, sem subfolders)
- **Wikilinks sem path absoluto**: prefira `[[Nome da Nota]]` em vez de `[[Pasta/Nome da Nota]]`
- **Idioma das seções autorais** de Glosas (TL;DR, Pontos-chave, Meu comentário, Ver também): PT-BR
- **Idioma das citações**: língua original do artigo

## Templates

Veja `00-Meta/templates/`. O `Template - Glosa.md` é usado tanto pela skill (gerando o arquivo) quanto manualmente via Templater no Obsidian (criação manual).
