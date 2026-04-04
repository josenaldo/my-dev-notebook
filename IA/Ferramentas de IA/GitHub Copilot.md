---
title: "GitHub Copilot"
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

# GitHub Copilot

Assistente de código integrado ao IDE — code completion, chat, e agent mode.

## O que é

GitHub Copilot é um assistente de IA para desenvolvimento, integrado diretamente em VS Code, JetBrains e outros IDEs. Usa modelos da OpenAI (GPT-4, Codex) e Anthropic (Claude) para code completion em tempo real, chat contextual, e recentemente agent mode para tarefas multi-step.

## Funcionalidades

- **Code Completion:** sugestões inline enquanto digita (ghost text)
- **Copilot Chat:** conversa no IDE com contexto do código aberto
- **Copilot Agent Mode:** executa tarefas multi-step (editar, rodar, testar)
- **Copilot for PRs:** resume mudanças, sugere reviewers
- **Custom instructions:** personalizar comportamento por projeto

## Diferenciais

- **Integração nativa no IDE:** menor friction que copiar/colar de chat
- **Contexto automático:** usa arquivos abertos, imports, e projeto como contexto
- **Velocidade:** completions em milissegundos
- **Ecossistema GitHub:** integração com Issues, PRs, Actions

## Quando usar

- **Code completion:** código repetitivo, boilerplate, patterns conhecidos
- **Chat:** explicar código, debugging, gerar testes
- **Agent mode:** implementar features com múltiplas mudanças de arquivos

## Na prática (da minha experiência)

> No MedEspecialista, configurei GitHub Copilot com instructions customizadas (skills e prompts) para o contexto do projeto. Isso permite que o completion entenda nosso design system, padrões de API, e convenções. Para code review, Copilot identifica patterns comuns de bugs e sugere melhorias.

## Recursos

- [Copilot Customization Overview](https://code.visualstudio.com/docs/copilot/customization/overview)
- [Awesome Copilot](https://github.com/github/awesome-copilot)
- [Como reduzi em 70% o uso do context window](https://www.linkedin.com/pulse/como-reduzi-em-70-o-uso-do-context-window-github-copilot-lemos-cycnf/)

## Veja também

- [[Comparativo de LLMs]]
- [[Claude]]
- [[Codex]]
- [[Skills e Prompting]]
