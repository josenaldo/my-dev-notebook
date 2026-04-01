---
title: "Claude"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - ia
  - llm
  - ferramentas
publish: false
---

# Claude

LLM da Anthropic — forte em raciocínio, código, análise longa, e segurança.

## O que é

Claude é a família de LLMs da Anthropic. Destaca-se por: raciocínio profundo, context window grande (até 1M tokens), forte em código e análise, e foco em segurança (Constitutional AI). Disponível como API, web (claude.ai), e CLI (Claude Code).

## Modelos

| Modelo | Força | Context | Use case |
| --- | --- | --- | --- |
| Claude Opus 4 | Mais capaz, raciocínio complexo | 1M tokens | Arquitetura, análise profunda, coding complexo |
| Claude Sonnet 4 | Equilíbrio performance/custo | 200K tokens | Coding diário, code review, análise |
| Claude Haiku 3.5 | Rápido e barato | 200K tokens | Classificação, extração, tarefas simples |

## Ferramentas do ecossistema

- **Claude.ai:** interface web e mobile
- **Claude Code:** CLI/IDE para coding assistido. Edita arquivos, roda testes, faz commits.
- **Claude API:** integração programática via SDK (Python, TypeScript)
- **Claude Agent SDK:** framework para construir agents customizados
- **MCP (Model Context Protocol):** protocolo para conectar Claude a fontes de dados externas

## Diferenciais

- **Context window 1M:** pode processar codebases inteiros, documentos longos
- **Tool Use nativo:** o modelo decide quando e como usar ferramentas
- **Extended Thinking:** raciocínio step-by-step antes de responder (similar a CoT)
- **Artifacts:** geração de documentos, código, diagramas na interface web
- **Constitutional AI:** treinado para ser útil, honesto e inofensivo

## Na prática (da minha experiência)

> Uso Claude Code diariamente como pair programmer. Para tarefas complexas como refatoração de arquitetura ou design de APIs, Claude Opus com contexto longo é essencial — consigo carregar toda a codebase relevante. Para tarefas rotineiras de código, Sonnet oferece o melhor equilíbrio de qualidade e velocidade.

## Recursos

- [Claude Docs](https://docs.anthropic.com/) — documentação oficial
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

## Veja também

- [[Comparativo de LLMs]]
- [[Agents]]
- [[LLMs]]
