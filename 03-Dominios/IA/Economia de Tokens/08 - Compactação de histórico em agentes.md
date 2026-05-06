---
title: "Compactação de histórico em agentes"
created: 2026-05-02
updated: 2026-05-02
type: concept
status: seedling
publish: true
tags:
  - economia-tokens
  - ia
  - custos
aliases:
  - Context compaction
  - History summarization
---

# Compactação de histórico em agentes

> [!abstract] TL;DR
> Em sessões longas, o histórico cresce até explodir o contexto e o custo. Compactação substitui turns antigos por resumos densos. Claude Code faz isso automaticamente via `/compact`. Para agentes custom, use rolling summarization ou anchored state documents. Mantém os últimos 5-10 turns completos e sumariza o resto.

## Como funciona

### Rolling summarization

Manter últimos N turns completos + sumarizar blocos anteriores em ~2k tokens cada.

### Anchored state document

Manter um "session state" continuamente atualizado com: objetivo, decisões, artefatos, problemas pendentes.

### Observation masking

Remover turns de baixo valor: retries falhados, leituras de arquivos já modificados, explicações redundantes.

### Impacto

| Sessão 50 turns | Sem compactação | Com compactação |
| --------------- | --------------- | --------------- |
| Input total     | ~2M tokens      | ~500k tokens    |
| Custo (Sonnet)  | ~$6.00          | ~$1.50          |

## Armadilhas

- Sumarizar sem preservar decisões de design causa contradições futuras
- Compactar turns recentes perde contexto útil — mantenha os últimos 5-10 completos

## Veja também

- [[06 - Context pruning — o que remover do prompt]]
- [[09 - Model routing — modelo certo para a tarefa]]

## Referências

- **Letta (MemGPT)** — *Tiered Memory Architecture* (2026).
- **Anthropic** — *Claude Code Context Management* (2026).
