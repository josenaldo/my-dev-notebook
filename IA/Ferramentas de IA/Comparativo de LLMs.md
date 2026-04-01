---
title: "Comparativo de LLMs"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - ia
  - llm
  - entrevista
publish: false
---

# Comparativo de LLMs

Diferenças entre Claude, GPT, Gemini e ferramentas de coding — quando usar cada um.

## Modelos (famílias principais)

| Família | Empresa | Modelos principais | Context max | Força |
| --- | --- | --- | --- | --- |
| Claude | Anthropic | Opus 4, Sonnet 4, Haiku 3.5 | 1M tokens | Raciocínio, código, análise longa, segurança |
| GPT | OpenAI | GPT-4.1, o1, o3 | 1M tokens | Versatilidade, ecossistema amplo |
| Gemini | Google | 2.5 Pro, 2.5 Flash | 2M tokens | Multimodal nativo, integração Google |

## Ferramentas de coding

| Ferramenta | Modelo base | Tipo | Ambiente | Ponto forte |
| --- | --- | --- | --- | --- |
| Claude Code | Claude (Anthropic) | CLI/IDE interativo | Local | Raciocínio, contexto longo, pair programming |
| GitHub Copilot | GPT + Claude | IDE inline + chat | IDE | Code completion, integração GitHub |
| Codex (OpenAI) | o1, GPT-4.1 | Agent cloud | Sandbox | Tarefas assíncronas, PRs automáticos |
| Gemini CLI | Gemini (Google) | CLI | Local | Multimodal, integração GCP |
| Cursor | Múltiplos | IDE completo | IDE | IDE AI-first, tab completion |

## Quando usar cada modelo

### Para coding

- **Claude (Opus/Sonnet):** refatoração complexa, design de arquitetura, code review profundo, debugging difícil
- **GPT-4.1 / o1:** coding geral, quando precisa de raciocínio step-by-step (o1)
- **Gemini 2.5 Pro:** análise de código com contexto muito grande, projetos com imagens/diagramas

### Para ferramentas de dev

- **Claude Code:** quando quer pair programming interativo, conversação contextual, edições locais
- **GitHub Copilot:** code completion no IDE, produtividade no dia a dia
- **Codex:** tarefas bem definidas que podem rodar em background
- **Cursor:** quando quer um IDE inteiro centrado em IA

### Para aplicações

- **Claude API:** chatbots, análise de documentos, classificação de texto, tool use
- **OpenAI API:** versatilidade, maior ecossistema de integrações
- **Gemini API:** multimodal (imagem + texto), grounding com busca Google

## Trade-offs

| Aspecto | Claude | GPT | Gemini |
| --- | --- | --- | --- |
| Raciocínio profundo | Forte | Forte (o1/o3) | Bom |
| Código | Muito forte | Forte | Bom |
| Multimodal | Texto + imagem | Texto + imagem + áudio | Nativo (tudo) |
| Context window | 1M | 1M | 2M |
| Velocidade | Boa | Boa | Flash é muito rápido |
| Custo | Moderado | Moderado | Competitivo |
| Segurança/Alinhamento | Forte (Constitutional AI) | Bom | Bom |
| Ecossistema | Claude Code, MCP | Copilot, ChatGPT, Codex | Google Cloud, Search |

## Minha abordagem

> Uso Claude Code como ferramenta principal de desenvolvimento (pair programming, refatoração, code review). GitHub Copilot para completions no IDE. Para aplicações, escolho o modelo baseado no use case: Claude para análise e raciocínio, OpenAI quando o ecossistema de integrações importa, Gemini quando preciso de multimodal ou contexto muito longo.

## How to explain in English

"I use multiple AI tools and choose based on the task. For day-to-day coding, GitHub Copilot provides fast inline completions. For complex tasks like architecture design or deep code review, I use Claude Code because of its strong reasoning and large context window.

The key differences are: Claude excels at reasoning and code quality, GPT models have the broadest ecosystem and versatility, and Gemini stands out for native multimodal capabilities and the largest context window.

For building AI features in applications, I choose the model based on requirements: Claude for tasks that need deep understanding, OpenAI for broad compatibility, and Gemini when the task involves processing images, audio, or very long documents."

### Key vocabulary

- modelo multimodal → multimodal model
- janela de contexto → context window
- completação de código → code completion
- agent de código → coding agent
- raciocínio → reasoning
- grounding → grounding: ancoragem em dados reais

## Veja também

- [[Claude]]
- [[GitHub Copilot]]
- [[Codex]]
- [[Gemini]]
- [[Agents]]
- [[LLMs]]
