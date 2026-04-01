---
title: "Gemini"
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

# Gemini

Família de LLMs do Google — multimodal nativo, integrado ao ecossistema Google.

## O que é

Gemini é a família de modelos de IA do Google DeepMind. Destaca-se por ser multimodal nativo (texto, imagem, áudio, vídeo desde o treinamento), context window grande (até 2M tokens no Gemini 2.5 Pro), e integração profunda com o ecossistema Google (Search, Workspace, Cloud).

## Modelos

| Modelo | Força | Context | Use case |
| --- | --- | --- | --- |
| Gemini 2.5 Pro | Mais capaz, multimodal | 1M-2M tokens | Análise complexa, multimodal |
| Gemini 2.5 Flash | Rápido, eficiente | 1M tokens | Tarefas do dia a dia |
| Gemini 2.0 Flash Lite | Ultra-rápido, barato | 128K tokens | Classificação, extração |

## Diferenciais

- **Multimodal nativo:** treinado desde o início com texto, imagem, áudio e vídeo
- **Context window 2M:** o maior entre os principais LLMs
- **Google Search Grounding:** pode buscar na web durante a geração
- **Integração Google:** Workspace, Cloud, Android, Chrome
- **Gemini CLI:** ferramenta de linha de comando para coding (similar ao Claude Code)

## Ferramentas

- **Gemini (web/app):** interface conversacional em gemini.google.com
- **Gemini API:** via Google AI Studio ou Vertex AI
- **Gemini CLI:** agent de código no terminal
- **Gemini in IDE:** extensão para VS Code via Google Cloud Code

## Quando usar

- **Análise multimodal:** quando precisa processar imagens, vídeos ou áudio junto com texto
- **Contexto muito longo:** documentos com mais de 1M tokens
- **Integração Google Cloud:** projetos já no ecossistema GCP
- **Grounding com busca:** quando respostas precisam de dados atuais da web

## Veja também

- [[Comparativo de LLMs]]
- [[Claude]]
- [[LLMs]]
