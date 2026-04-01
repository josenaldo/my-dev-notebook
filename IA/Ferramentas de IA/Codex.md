---
title: "Codex"
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

# Codex

Agent de código da OpenAI — executa tarefas de desenvolvimento em ambiente sandboxed.

## O que é

Codex (OpenAI, 2025) é um coding agent cloud-based que executa tarefas de software em ambientes sandboxed. Diferente do Copilot (completions no IDE), Codex opera como um agent autônomo que pode clonar repos, editar código, rodar testes e criar PRs.

## Como funciona

- Recebe uma tarefa em linguagem natural
- Executa em ambiente isolado (sandbox) com acesso ao repositório
- Pode rodar comandos, editar arquivos, rodar testes
- Retorna resultado como diff/PR

## Diferenciais

- **Sandboxed execution:** seguro, isolado, não afeta o ambiente local
- **Async tasks:** pode processar múltiplas tarefas em paralelo
- **GitHub integration:** cria PRs automaticamente
- **Baseado em modelos OpenAI:** o1, GPT-4.1

## Quando usar

- **Tarefas bem definidas:** "adicione validação de email no formulário X"
- **Refatoração em lote:** "renomeie todos os métodos de camelCase para snake_case"
- **Bug fixes com contexto claro:** "corrija o bug #123"
- **Geração de testes:** "adicione testes unitários para o módulo Y"

## Comparação com Claude Code

| Aspecto | Codex | Claude Code |
| --- | --- | --- |
| Execução | Cloud (sandbox) | Local (seu terminal) |
| Interação | Assíncrono | Interativo (conversa) |
| Contexto | Repo clonado | Codebase local + conversa |
| Output | PR/diff | Edições diretas + commits |
| Modelo | OpenAI (o1, GPT-4.1) | Anthropic (Claude Opus/Sonnet) |

## Veja também

- [[Comparativo de LLMs]]
- [[Claude]]
- [[GitHub Copilot]]
- [[Agents]]
