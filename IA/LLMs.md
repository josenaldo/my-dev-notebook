---
title: "LLMs"
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

# LLMs

Large Language Models — como funcionam, como usar, e o que todo dev precisa saber.

## O que é

LLMs são redes neurais treinadas em grandes volumes de texto para prever o próximo token. O resultado é um modelo que "entende" linguagem e pode gerar texto, código, análises e raciocínio. Para devs, o foco é em como usar via API, otimizar prompts, e entender limitações.

## Como funciona

### Arquitetura (simplificada)

```text
Input (texto/prompt)
  → Tokenização (texto → tokens numéricos)
  → Transformer (atenção sobre toda a sequência)
  → Predição (probabilidade do próximo token)
  → Decodificação (tokens → texto)
Output (resposta)
```

**Transformer:** arquitetura base (Google, 2017). Self-attention permite que cada token "olhe" para todos os outros tokens no contexto.

### Parâmetros-chave de API

| Parâmetro | O que faz | Valor típico |
| --- | --- | --- |
| `temperature` | Aleatoriedade (0=determinístico, 1=criativo) | 0-0.7 para código, 0.7-1 para criativo |
| `max_tokens` | Limite de tokens na resposta | Depende do use case |
| `top_p` | Nucleus sampling (alternativa a temperature) | 0.9-1.0 |
| `system` | Instrução de sistema/persona | Define comportamento base |
| `stop` | Sequências que param a geração | `["\n\n"]`, `["```"]` |

### Tokenização

- 1 token ≈ 4 caracteres (EN) / ≈ 3 caracteres (PT-BR)
- "Hello world" = 2 tokens
- "Inteligência Artificial" = ~5 tokens
- Código geralmente usa mais tokens (sintaxe, indentação)

### Limitações fundamentais

- **Hallucination:** gerar informação plausível mas incorreta. Mais comum com fatos específicos.
- **Knowledge cutoff:** modelo não sabe de eventos após a data de treino.
- **Context window:** limite de tokens por conversa. Exceder = perda de informação.
- **Não-determinístico:** mesma pergunta pode gerar respostas diferentes (temperature > 0).
- **Viés:** reflete vieses dos dados de treinamento.

## Quando usar

- **Text generation:** chatbots, sumarização, tradução, reescrita
- **Code generation:** completar, refatorar, gerar testes, explicar código
- **Classification:** categorizar texto, análise de sentimento
- **Extraction:** extrair dados estruturados de texto livre
- **RAG:** combinar com busca vetorial para respostas baseadas em dados privados

## Armadilhas comuns

- **Prompts vagos:** "me ajude com código" vs "gere uma função TypeScript que valida CPF usando o algoritmo de dígitos verificadores"
- **Ignorar o system prompt:** é a ferramenta mais poderosa para controlar comportamento
- **Context window overflow:** em conversas longas, informação antiga é "esquecida"
- **Não validar output:** especialmente código — sempre testar

## How to explain in English

"LLMs are the technology behind tools like Claude and GPT. They're transformer-based neural networks trained to predict the next token in a sequence. What makes them useful for developers is their ability to understand and generate both natural language and code.

When integrating LLMs via API, the key parameters are temperature for controlling creativity, system prompts for defining behavior, and max_tokens for managing response length. For production applications, I focus on prompt engineering to get consistent, reliable outputs and implement RAG when the model needs access to private or current data.

Understanding token economics is important — each API call costs based on tokens processed, so efficient prompts and caching strategies directly impact the bottom line."

### Key vocabulary

- modelo de linguagem grande → large language model (LLM)
- transformador → transformer: arquitetura de rede neural
- tokenização → tokenization: dividir texto em tokens
- janela de contexto → context window
- alucinação → hallucination
- engenharia de prompt → prompt engineering
- amostragem → sampling: como tokens são selecionados

## Veja também

- [[Inteligência Artificial]]
- [[Agents]]
- [[Skills e Prompting]]
- [[Comparativo de LLMs]]
