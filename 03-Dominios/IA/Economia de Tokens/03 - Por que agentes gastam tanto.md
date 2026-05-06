---
title: "Por que agentes gastam tanto"
created: 2026-05-02
updated: 2026-05-02
type: concept
status: seedling
publish: true
tags:
  - economia-tokens
  - ia
  - custos
  - agentes
aliases:
  - Agent token cost
  - Por que agente custa caro
  - Loop agentic cost
---

# Por que agentes gastam tanto

> [!abstract] TL;DR
> Uma chamada single-shot de LLM custa centavos. Uma sessão de agente custa dólares. A diferença é estrutural, não acidental: o loop agentic re-envia contexto a cada turno (acumulação quadrática), tool definitions ficam infladas no system prompt, retries silenciosos consomem tokens sem feedback visível, e o agente pode entrar em rabbit holes iterando sem progresso. Entender essa dinâmica é pré-requisito para qualquer otimização real.

## Os cinco vetores de gasto

### 1. Acumulação de contexto turno-a-turno

Cada turno do loop agentic envia **todo o histórico** mais a nova mensagem. Sessão de 30 turnos onde cada turno acrescenta 1K tokens:

| Turno | Tokens enviados (input) | Acumulado |
|---|---|---|
| 1 | 1K | 1K |
| 5 | 5K | 15K |
| 15 | 15K | 120K |
| 30 | 30K | 465K |

Sem [[05 - Prompt caching na prática]], cada token desse histórico é cobrado como input fresco.

### 2. Tool definitions infladas

Tool descriptions são re-enviadas no system prompt **a cada turno**. Um conjunto típico de 15 ferramentas com schemas detalhados consome 5-15K tokens. Multiplicado por 30 turnos: 150-450K tokens só em definição de tools — antes de o agente fazer qualquer coisa útil.

Ver [[07 - Compressão de tool definitions]].

### 3. Tool outputs verbosos

O modelo lê *toda* a saída de cada tool. Casos comuns:

- `bash: npm install` → 2-5K tokens de log
- `read_file` em arquivo grande → 10-50K tokens
- `grep` sem filtro → centenas de matches
- Stack traces e erros completos quando bastaria a primeira linha

Cada output verboso vira input do próximo turno. O modelo já leu — mas o histórico acumula igual.

### 4. Retries silenciosos

Quando o modelo erra a sintaxe de uma tool call, frameworks geralmente tentam de novo automaticamente. Cada retry custa um turno completo (input acumulado + nova geração). Em logs típicos do Claude Code ou Cursor agent, **5-15% dos turnos são retries** — invisíveis para o usuário.

### 5. Rabbit holes

Agentes podem iterar sem fazer progresso real:

- Tentar 4 abordagens diferentes antes de admitir que precisa do humano
- Investigar recursivamente sem encontrar a causa raiz
- Reescrever o mesmo arquivo várias vezes
- Loops de "verificar → ajustar → verificar" sem critério de parada

Sem [[15 - Orçamento e hard limits|kill switches]], um rabbit hole pode queimar 200K+ tokens em uma única sessão.

## Single-shot vs agente — comparação concreta

> [!example] "Adicione validação de email a este formulário"
> | Modo | Tokens input | Tokens output | Custo (Sonnet 4.6) |
> |---|---|---|---|
> | Single-shot (paste do código) | 2K | 800 | ~$0.02 |
> | Agente sem otimização | 180K | 4K | ~$0.65 |
> | Agente otimizado (caching + pruning) | 45K efetivos | 4K | ~$0.12 |
>
> Diferença de **30x** entre single-shot e agente cru. Diferença de **5x** entre agente cru e agente otimizado.

## Quando o gasto é justificado

Agente custa mais — mas pode entregar mais. Vale o gasto extra quando:

- Tarefa exige **múltiplos arquivos** (single-shot exigiria copiar manualmente)
- Tarefa exige **execução** (rodar testes, ler logs, iterar)
- Especificação está **incompleta** e precisa de exploração

Não vale quando:

- Tarefa cabe em uma única caixa de chat
- Você já sabe o que mudar e onde
- O custo de erro é alto e validação humana é necessária por turno

## Veja também

- [[02 - Anatomia do gasto — input, output e reasoning]]
- [[05 - Prompt caching na prática]]
- [[07 - Compressão de tool definitions]]
- [[08 - Compactação de histórico em agentes]]
- [[15 - Orçamento e hard limits]]

## Referências

- **Anthropic** — *Building effective agents* (2025).
- **Latent Space Pod** — *The economics of agent loops* (2025).
