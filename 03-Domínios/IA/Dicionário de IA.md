---
title: "Dicionário de IA"
created: 2026-05-03
updated: 2026-05-03
type: glossary
status: seedling
aliases:
  - AI Glossary
  - Glossário de IA
tags:
  - glossary
  - ia
lang: en
publish: true
---

# Dicionário de IA

> Working glossary for the IA domain — LLMs, agents, RAG, MCP, context engineering, and the surrounding ecosystem. Definitions in English, with PT annotations when a term has a strong Portuguese counterpart in active use.

<!--
Como usar:
- Cada verbete é um `###` dentro de uma `##` temática.
- Linkar: [[Dicionário de IA#RAG]]
- Adicionar termos: use a skill /verbete (auto-pesquisa se faltar definição).
- Os bullets `- TODO:` em cada seção são candidatos a verbetes; promova conforme estudar.
-->

## Agents and Agentic Systems

### Agent
A program that uses an LLM in a loop to take actions toward a goal: it observes state, decides on a tool call or response, executes, and feeds the result back into the next step. The defining property is autonomy across multiple turns, not raw intelligence.

- TODO: agentic loop
- TODO: orchestrator-worker
- TODO: planning
- TODO: ReAct
- TODO: tool use
- TODO: subagent

## Coding Agents

### Coding agent
An agent specialized in software engineering tasks — reading, writing, and modifying code, executing shell commands, running tests, and iterating until a goal is met. Examples include Claude Code, Cursor, Aider, and Continue.

- TODO: Aider
- TODO: Claude Code
- TODO: Continue
- TODO: Cursor
- TODO: autonomous coding loop
- TODO: PR-driven workflow

## Context Engineering

### Context window
The maximum number of tokens a model can attend to in a single inference call, including system prompt, user input, prior turns, tool definitions, and the response being generated. Exceeding it forces truncation, summarization, or compaction.

- TODO: chain-of-thought
- TODO: context compaction
- TODO: few-shot prompting
- TODO: prompt engineering
- TODO: prompt template
- TODO: system prompt

## LLMs Anatomy

### LLM (Large Language Model)
A neural network — typically a decoder-only transformer — trained on large text corpora to predict the next token given a sequence. Modern LLMs scale to billions or trillions of parameters and exhibit emergent capabilities like in-context learning and instruction following.

- TODO: attention
- TODO: decoding strategy
- TODO: embedding
- TODO: fine-tuning
- TODO: inference
- TODO: parameters / weights
- TODO: sampling
- TODO: temperature
- TODO: top-k
- TODO: top-p
- TODO: transformer

## MCP — Model Context Protocol

### MCP (Model Context Protocol)
An open protocol that standardizes how LLM applications expose context, tools, and prompts to models through a client-server architecture. It decouples model providers from data sources, letting any MCP-compatible client connect to any MCP server.

- TODO: MCP client
- TODO: MCP server
- TODO: prompts (MCP)
- TODO: resources (MCP)
- TODO: tools (MCP)
- TODO: transport (stdio, SSE, HTTP)

## Memory

- TODO: episodic memory
- TODO: long-term memory
- TODO: recall
- TODO: semantic memory
- TODO: vector store
- TODO: working memory

## RAG and Vector Databases

### RAG (Retrieval-Augmented Generation)
A technique that grounds LLM responses in external documents fetched at query time, reducing hallucination and enabling knowledge updates without retraining. A typical pipeline embeds the query, retrieves the top-K relevant chunks from a vector store, and injects them into the prompt.

- TODO: BM25
- TODO: chunking
- TODO: dense retrieval
- TODO: embedding model
- TODO: hybrid search
- TODO: reranking
- TODO: retrieval
- TODO: vector database

## Security and Guardrails

### Guardrail
A constraint applied to LLM input or output to enforce safety, policy, or quality requirements — e.g., blocking PII, filtering harmful content, validating structured output schemas, or refusing off-topic requests. Guardrails can be model-side (fine-tuning, system prompt) or pipeline-side (pre/post processing).

- TODO: content filtering
- TODO: jailbreak
- TODO: output validation
- TODO: prompt injection
- TODO: red teaming

## Sequence Models

### LSTM (Long Short-Term Memory)
A recurrent neural network architecture introduced by Hochreiter & Schmidhuber (1997) that uses input, forget, and output gates to maintain information across long sequences, mitigating the vanishing gradient problem of vanilla RNNs. Dominated sequence modeling tasks like translation and speech recognition before being largely displaced by the Transformer.

## Spec-Driven Development

### Spec-driven development
A workflow where a written specification (requirements + design) precedes implementation, and the spec — not just the code — is the artifact reviewed and iterated on. With AI assistants, the spec also becomes the input that drives plan generation and code synthesis.

- TODO: brainstorming (process)
- TODO: design doc
- TODO: implementation plan
- TODO: TDD with AI

## Token Economy

### Token
The atomic unit a language model reads and emits — typically a sub-word fragment produced by a tokenizer. Pricing, context limits, and latency are all measured in tokens, so understanding tokenization is foundational to cost and performance optimization.

- TODO: batch API
- TODO: cache hit rate
- TODO: completion tokens
- TODO: cost per token
- TODO: prompt caching
- TODO: prompt tokens

## Tooling

- TODO: function calling
- TODO: SDK
- TODO: structured output
- TODO: tool definition
