---
title: "Sendas e Domínios — modelo canônico"
type: moc
publish: true
created: 2026-05-03
updated: 2026-05-03
tags:
  - guia
  - vault
  - canon
---

# Sendas e Domínios — modelo canônico

Este documento define como notas, domínios e sendas se relacionam no Codex.

## As quatro zonas

| Zona | Papel |
|---|---|
| `01-Pergaminhos/` | Captura inicial: inbox, links soltos, ideias cruas. |
| `02-Glosas/` | Fichamentos de materiais externos consumidos. |
| `03-Domínios/` | Corpus do conhecimento. **Centro gravitacional do vault**. Notas amadurecidas, organizadas por domínio. |
| `04-Sendas/` | Mapas de leitura. Wikilinks ordenados pra notas dos domínios. **Apenas isso**. |

**Pipeline de alimentação:** pergaminho → glosa → nota de domínio.

## Domínios = estantes; Sendas = fluxograma

- **Domínios** são estantes de livros: corpus de conhecimento sobre um tema. Notas, materiais externos contextualizados (via callout `[!convite]`), exercícios, projetos — tudo aqui.
- **Sendas** são fluxogramas de faculdade: indicam a ordem de leitura das notas (livros) das estantes. Não contêm conhecimento próprio; apenas referenciam.

### Uma senda **não** contém

- Texto explicativo de conceitos (vai pra nota do domínio).
- Links externos a artigos, vídeos, livros, cursos (vão pra notas de domínio, dentro de callouts `[!convite]`, ou viram glosas).
- Exercícios, projetos, checkpoints conceituais (vão pro domínio).
- Wikilinks pra headings internos do próprio arquivo.

### Uma senda **contém**

- Frontmatter com metadados.
- Descrição curta da senda (pra quem é, qual o destino).
- Pré-requisitos (wikilinks pra notas de outros domínios).
- Sequência ordenada de wikilinks pra notas do domínio principal (forma plana ou em fases).
- Bloco dataview de agregação de progresso.

## Organização dos domínios (modelo de estantes)

Cada domínio é uma estante coerente: uma área de conhecimento com vocabulário próprio, comunidade própria, ferramental próprio.

- **Linguagens próprias** ganham estante própria: `JavaScript/` (apenas core), `TypeScript/`, `HTML/`, `CSS/`, `Java/`, `Python/`, `Go/`.
- **Bibliotecas/frameworks grandes** ganham estante própria: `React/`, `Vue/`, `Svelte/`, `HTMX/`. Bibliotecas menores moram como notas dentro do domínio mais relevante.
- **Disciplinas (cross-tech)** ganham estante própria: `Frontend/` é a estante de **engenharia frontend** — arquitetura, performance, acessibilidade, padrões.
- **Ferramentas** vão pra `Ferramentas/` (tooling cross-tech: Vite, bundlers, monorepos).

Sendas atravessam estantes naturalmente.

## Campo `progresso` no frontmatter

Toda nota de domínio (e toda glosa) tem o campo `progresso`:

```yaml
progresso: pendente | andamento | feito | pausado | abandonado
```

| Valor | Significado |
|---|---|
| `pendente` | Ainda não comecei. Default quando o campo está ausente. |
| `andamento` | Estou estudando ativamente. |
| `feito` | Concluí o estudo dessa nota; conteúdo absorvido. |
| `pausado` | Comecei e parei intencionalmente; pretendo retomar. |
| `abandonado` | Decidi não estudar; fora do meu radar. |

`progresso` é **ortogonal** a `status` (maturidade da nota: `seedling`/`budding`/`evergreen`). São dois eixos independentes.

## Callout `[!convite]`

Materiais externos (artigos, vídeos, cursos) são apresentados dentro de notas de domínio como **convites a aprofundamento**:

```markdown
> [!convite] Aprofundamento — <tema>
> - [Texto do link](https://...) — autor/origem
> - Outro item
>
> Consumiu? Faça uma glosa em `02-Glosas/` e amadureça pro domínio quando fizer sentido.
```

Convite é convite: opcional, sem `progresso` próprio. Quando o usuário consome o material, cria uma glosa, e a glosa carrega o `progresso`.

## Ver também

- [[Como usar este vault]]
- [[Convenções de escrita]]
- [[Decisões do vault]]
- [[Wikilinks e MOCs]]
