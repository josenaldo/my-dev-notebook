---
title: "LLMs"
created: 2026-04-01
updated: 2026-04-11
type: concept
status: evergreen
tags:
  - ia
  - llm
  - entrevista
  - fundamentos
publish: true
---

# LLMs

> Large Language Models são a tecnologia que destravou tudo que chamamos de "IA" em 2025-2026. Para um senior dev, entender LLMs não é sobre treinar um GPT do zero — é sobre **como escolher modelo, usar a API corretamente, dominar os parâmetros, controlar custo, lidar com não-determinismo e desenhar sistemas robustos sobre uma peça estocástica**. Esta nota é a trilha: do "o que é um transformer" até "como operar LLMs em produção com confiança".

## O que é

Um **Large Language Model** é uma rede neural baseada em arquitetura Transformer, treinada em vastíssimos volumes de texto (e código) com um objetivo aparentemente simples: **prever o próximo token dado o contexto anterior**. Dessa tarefa emerge uma capacidade surpreendente de entender e gerar linguagem, seguir instruções, raciocinar (imperfeitamente), escrever código, traduzir, sumarizar, e interagir como assistente.

O "large" em LLM refere-se a:

- **Parâmetros:** de bilhões a trilhões. GPT-3 tinha 175B, Llama 3 70B, Claude 3.5/4 e GPT-4+ na casa dos trilhões.
- **Dados de treino:** trilhões de tokens — efetivamente a maior parte do texto de qualidade disponível na internet, livros, papers, código.
- **Compute:** treinos custam dezenas a centenas de milhões de dólares em GPUs.

Para você, dev fullstack, o que importa é:

1. **Como LLMs funcionam por dentro o suficiente para tomar boas decisões** (não para re-implementar).
2. **Como usar a API com competência** — parâmetros, streaming, tool use, structured outputs, caching.
3. **Como navegar as limitações** — custo, latência, não-determinismo, alucinação, context rot.
4. **Como escolher entre modelos** e quando trocar.

## O que diferencia um senior em LLMs

1. **Tem modelo mental correto de tokenização e consegue estimar tokens de cabeça** (~4 chars/token EN, ~3 PT-BR, mais para código).
2. **Distingue pretraining, SFT e RLHF** e sabe explicar por que um modelo recusa/aceita certos prompts.
3. **Entende attention e context window de forma prática** — sabe por que "lost in the middle" acontece e como mitigar.
4. **Usa temperature, top_p, top_k com intenção,** não como magia.
5. **Desenha prompts que retornam JSON válido consistentemente** via structured outputs + validação + retry.
6. **Sabe quando usar streaming vs não** (UX vs facilidade de parsing).
7. **Aplica prompt caching corretamente** e mede impacto em custo.
8. **Faz tiering de modelos** — roteia para Haiku/Flash o que der, escala para Opus/GPT-4 o necessário.
9. **Mede tudo:** tokens in/out, latência, custo, taxa de erro. LLM sem observabilidade é caixa preta.
10. **Trata LLM como fonte não-confiável** — valida, faz retry, tem fallback, não confia em "parece ok".

## Trilha de aprendizado — zero até domínio de LLMs

### Nível 1 — Intuição (3-5 dias)

- Use ChatGPT/Claude/Gemini pesadamente por uma semana em problemas reais.
- Observe quando acertam, quando alucinam, quando são chatos demais.
- Leia [The Illustrated Transformer — Jay Alammar](https://jalammar.github.io/illustrated-transformer/).
- Assista [3Blue1Brown — But what is a GPT?](https://www.3blue1brown.com/lessons/gpt).

**Check:** você consegue explicar "atenção" para um colega usando analogia concreta?

### Nível 2 — Conceitos técnicos (2-3 semanas)

- Tokenização, embeddings, context window, cutoff.
- Parâmetros da API: temperature, top_p, max_tokens, stop sequences, system prompt.
- Pretraining → SFT → RLHF (entender o porquê dos comportamentos).
- Limitações: hallucination, knowledge cutoff, não-determinismo, context rot.
- Leitura: [State of GPT — Andrej Karpathy](https://www.youtube.com/watch?v=bZQun8Y4L2A) (~40 min, clássico).

**Check:** você consegue ler a resposta de um LLM e ter intuição do porquê ele respondeu assim.

### Nível 3 — API hands-on (3-4 semanas)

- Construa 3-4 scripts com Claude API e OpenAI API: chat, streaming, tool use, structured outputs.
- Implemente golden set e evaluation para um prompt próprio.
- Compare latência e custo entre 3 modelos no mesmo problema.
- Implemente prompt caching e meça redução de custo.

**Check:** você consegue levantar uma integração LLM em produção confiando no que está fazendo.

### Nível 4 — Produção (1-2 meses)

- Observabilidade: Langfuse/Helicone/LangSmith integrados.
- Guardrails: PII detection, output filtering, prompt injection defense.
- Retry, fallback, timeout, circuit breaker.
- Rate limits e backoff.
- Cost management: budgets, alertas, tiering.

**Check:** você assume ownership de um serviço LLM-backed com confiança.

### Nível 5 — Avançado (ongoing)

- Fine-tuning (quando justificado): LoRA, QLoRA, DPO.
- Rodar LLMs locais: Ollama, LM Studio, vLLM.
- Quantização, distillation, model serving.
- Avaliação profunda: LLM-as-judge, human eval, A/B test.
- Leitura de papers (Attention is All You Need, InstructGPT, LLaMA, etc).

## Arquitetura — como um LLM funciona por dentro

### O pipeline de inferência, passo a passo

```text
User message (texto)
  │
  ▼
┌──────────────────┐
│ 1. Tokenização   │  texto → IDs numéricos (ex: "Hello" → [15496])
└──────────────────┘
  │
  ▼
┌──────────────────┐
│ 2. Embedding     │  IDs → vetores densos (ex: 15496 → [0.12, ..., -0.04])
└──────────────────┘
  │
  ▼
┌──────────────────┐
│ 3. Transformer   │  N blocos de (self-attention + FFN + residual + norm)
│    stack         │  cada token "olha" para todos os anteriores
└──────────────────┘
  │
  ▼
┌──────────────────┐
│ 4. LM Head       │  projeta para vocab: logits (1 número por token do vocab)
└──────────────────┘
  │
  ▼
┌──────────────────┐
│ 5. Sampling      │  escolhe próximo token (temperature, top_p, top_k)
└──────────────────┘
  │
  ▼
Token gerado
  │
  └──> adiciona ao contexto, volta para passo 3 até parar
```

### Tokenização — a unidade mínima

Tokens não são palavras. Tokenizers modernos (BPE — byte-pair encoding, usado por GPT, Claude, Llama) dividem o texto em **sub-palavras frequentes**.

Exemplos:

```text
"The quick brown fox"      → ["The", " quick", " brown", " fox"]           (4 tokens)
"Inteligência Artificial"  → ["Int", "el", "igê", "ncia", " Artificial"]   (~5 tokens)
"console.log('hello')"     → ["console", ".", "log", "('", "hello", "')"]  (~6 tokens)
"🦀"                       → ["\xf0", "\x9f", "\xa6", "\x80"]              (4 tokens — 1 por byte UTF-8)
```

**Por que importa:**

- **Custo:** APIs cobram por token. Texto em português custa ~30% mais que inglês pelo mesmo conteúdo.
- **Context window:** limite é em tokens, não em caracteres ou palavras.
- **Performance:** texto "estranho" (linguagens não-latinas, emojis, código bagunçado) consome mais tokens.
- **Prompt engineering:** palavras raras podem ser quebradas em pedaços, afetando como o modelo processa.

**Ferramentas para contar tokens:**

- [tiktoken (OpenAI)](https://github.com/openai/tiktoken)
- [anthropic-tokenizer-python](https://github.com/anthropics/anthropic-tokenizer) ou `client.messages.count_tokens()`
- [OpenAI Tokenizer web](https://platform.openai.com/tokenizer)

### Embeddings — texto como vetor

Cada token vira um vetor denso de centenas/milhares de dimensões. Tokens com significado parecido ficam próximos nesse espaço vetorial.

```text
"rei"     → [0.12, -0.88, 0.34, ..., 0.21]
"rainha"  → [0.15, -0.85, 0.39, ..., 0.19]  // perto de "rei"
"mesa"    → [-0.77, 0.23, 0.12, ..., 0.64]  // longe
```

Dentro do modelo, embeddings são aprendidas durante o treino — não são fixas. Os modelos de embedding para RAG (text-embedding-3, voyage-2, etc) são treinados especificamente para similaridade semântica.

### Transformer blocks — o cérebro

Cada bloco Transformer tem três componentes-chave:

1. **Multi-head self-attention:** cada token calcula "quanto devo prestar atenção em cada outro token no contexto". Isso é feito em paralelo em múltiplas "cabeças" que podem focar em coisas diferentes (sintaxe, semântica, referências).
2. **Feed-forward network (FFN):** processa cada posição individualmente, refina representações.
3. **Residual connections + layer norm:** estabilidade de treinamento e gradient flow.

Modelos modernos empilham dezenas a centenas desses blocos. Claude/GPT-4 têm dezenas de camadas com milhares de dimensões por token.

**Self-attention em 1 parágrafo:** cada token é transformado em três vetores — Query, Key, Value. A "pontuação de atenção" entre token i e token j é `Q_i · K_j` (produto escalar). Depois de softmax, esses scores pesam o Value de cada token. Resultado: cada posição de saída é uma mistura ponderada de todos os inputs, com pesos baseados em "relevância".

**Por que importa saber:**

- Entende por que context window longo é caro: complexidade O(n²) em memória e compute com attention tradicional (há variantes como sliding window, mas o mental model vale).
- Entende por que "lost in the middle" existe: distribuição de atenção favorece início e fim.
- Entende por que "cite exact words from the context" geralmente funciona: attention tende a copiar.

### Causal / decoder-only — o padrão dominante

LLMs modernos (GPT, Claude, Llama, Gemini) são **decoder-only com causal attention**: cada token só pode olhar para tokens anteriores, nunca futuros. Isso é o que permite geração autoregressive (gerar 1 token de cada vez).

Existem outras arquiteturas (encoder-only como BERT, encoder-decoder como T5) mais usadas em tarefas específicas, mas o "LLM moderno" que você usa no Claude/GPT/Gemini é decoder-only.

### Sampling — como o próximo token é escolhido

Depois do forward pass, o modelo produz um vetor de **logits** — uma pontuação para cada token do vocabulário (dezenas de milhares). Softmax converte logits em distribuição de probabilidade. Aí vem o sampling:

**Greedy (temperature 0):** sempre pega o token mais provável. Determinístico, repetitivo, bom para tarefas estruturadas.

**Temperature sampling:** divide logits por temperature antes de softmax. Temperature alta = distribuição mais uniforme = mais aleatório.

**Top-k:** considera apenas os k tokens mais prováveis. k=50 é comum.

**Top-p (nucleus):** considera os tokens cuja probabilidade acumulada chega a p. p=0.9 é comum. **Mais estável que top-k** porque se adapta à forma da distribuição.

**Regras práticas:**

- **Código, JSON, classificação, extração → temperature 0.** Determinístico, reprodutível.
- **Brainstorming, escrita criativa, ideação → 0.7-1.0.**
- **Use temperature OU top_p, não os dois.** Default da maioria das APIs: temperature=1, top_p=1.
- **min_p (mais novo):** token só é considerado se sua probabilidade ≥ p × prob do mais provável. Mais robusto em geração longa. Ainda não disponível em todas APIs.

## Parâmetros da API — o guia definitivo

```python
# Exemplo com Anthropic Claude API
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="You are a senior Python developer...",
    temperature=0.2,
    top_p=0.95,
    stop_sequences=["</answer>"],
    messages=[
        {"role": "user", "content": "..."}
    ]
)
```

| Parâmetro | O que faz | Quando ajustar |
| --- | --- | --- |
| `model` | Qual modelo usar | Sempre decisão consciente (qualidade vs custo) |
| `max_tokens` | Teto de tokens de SAÍDA | Sempre defina; evita geração infinita e custo surpresa |
| `system` | Instrução persistente de comportamento | Sempre. É a ferramenta mais poderosa para controlar LLM |
| `messages` | Histórico conversacional | Formato user/assistant alternado |
| `temperature` | Aleatoriedade (0-2) | 0 para estruturado, 0.7+ para criativo |
| `top_p` | Nucleus sampling | 0.9-1.0. Use no lugar de temperature para estabilidade |
| `stop_sequences` | Para geração ao encontrar | Para delimitar formatos (ex: "```", "</answer>") |
| `stream` | Recebe tokens à medida que são gerados | UX (feedback imediato), análise em streaming |
| `tools` | Ferramentas que o modelo pode chamar | Tool use / function calling |
| `tool_choice` | Forçar uso de ferramenta | `auto`, `any`, ou força uma específica |

### System prompt — a alavanca mais poderosa

O system prompt é onde você define persona, regras, formato, restrições. É persistente através da conversa e tem peso desproporcional no comportamento do modelo.

```text
system:
You are a senior backend engineer reviewing Java code.
- Focus on: concurrency bugs, resource leaks, security issues
- Always cite line numbers
- Be direct; skip pleasantries
- If code is good, say so briefly
- Output format: markdown with ## sections
```

Boas práticas:

- **Seja específico sobre formato de saída.** "Responda em JSON com schema X" é mais confiável que "responda de forma estruturada".
- **Use listas, não parágrafos.** Instruções estruturadas são seguidas melhor.
- **Diga o que fazer, não só o que evitar.** "Responda em português" > "não responda em inglês".
- **Coloque restrições críticas no início e no fim.** Attention favorece as bordas.

### Tool use / function calling

LLMs modernos podem ser instruídos a "chamar funções" declaradas. O modelo retorna um JSON dizendo qual função chamar e com quais argumentos; seu código executa e devolve o resultado.

```python
tools = [{
    "name": "get_weather",
    "description": "Get current weather for a city",
    "input_schema": {
        "type": "object",
        "properties": {
            "city": {"type": "string"},
            "units": {"type": "string", "enum": ["celsius", "fahrenheit"]}
        },
        "required": ["city"]
    }
}]

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "Qual o clima em São Paulo?"}]
)
# response.stop_reason == "tool_use"
# response.content contains tool_use block with name + input
```

Depois você executa `get_weather(city="São Paulo", units="celsius")`, retorna o resultado ao modelo, e ele gera a resposta final ao usuário.

**Pontos críticos:**

- **Descrição da ferramenta** é o que o modelo lê para decidir quando usar. Seja claro.
- **Schema da entrada** é validado — erros de schema são detectados.
- **Agents** são construídos em cima de tool use em loop.
- Ver [[Agents]] para deep dive em construção de agents.

### Structured outputs / JSON mode

Para tarefas que precisam retornar JSON válido, modernos modelos suportam **structured outputs** — uma garantia de que a saída seguirá um schema JSON fornecido.

```python
# OpenAI structured outputs
response = client.chat.completions.create(
    model="gpt-4.1",
    messages=[...],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "classification",
            "schema": {
                "type": "object",
                "properties": {
                    "category": {"type": "string", "enum": ["bug", "feature", "question"]},
                    "confidence": {"type": "number", "minimum": 0, "maximum": 1},
                    "reasoning": {"type": "string"}
                },
                "required": ["category", "confidence", "reasoning"]
            }
        }
    }
)
```

Na Anthropic, tool use com uma ferramenta única e `tool_choice` forçado tem função análoga.

**Por que usar:**

- Elimina casos de "quase JSON válido" que quebram parsers.
- Reduz drasticamente retries.
- Validação automática no servidor do provedor.

### Prompt caching — a otimização de custo mais subestimada

Anthropic e OpenAI oferecem **prompt caching**: você marca partes do prompt como cacheadas, e chamadas subsequentes com o mesmo prefixo têm desconto enorme (até 90%) e latência menor.

Casos de uso matadores:

- **System prompt grande repetido:** cacheia uma vez, usa em toda chamada.
- **Documentos longos em RAG onde o mesmo doc é consultado várias vezes.**
- **Few-shot examples estáticos.**

```python
# Claude prompt caching
response = client.messages.create(
    model="claude-sonnet-4-6",
    system=[
        {
            "type": "text",
            "text": "Very long system instructions and few-shot examples...",
            "cache_control": {"type": "ephemeral"}  # marca para cache
        }
    ],
    messages=[...]
)
```

**Impacto típico:** features com system prompt de 3-5K tokens sob caching tipicamente reduzem custo em ~80-90% e TTFT em ~30-40%. ROI imediato em qualquer workload com system prompt grande recorrente.

### Streaming vs non-streaming

**Streaming:** recebe tokens à medida que são gerados (Server-Sent Events). Primeira letra aparece em ~300ms em vez de esperar a resposta completa.

**Quando usar:**

- UX conversacional (chatbot): sempre.
- Geração longa: sempre (percepção de responsividade).
- Tarefas curtas estruturadas (classificação, extração): não precisa.

**Pitfalls de streaming:**

- Parser precisa lidar com chunks.
- Error handling é mais complexo (erro pode aparecer no meio).
- Structured outputs em streaming exige parser incremental (alguns libs ajudam).

## Como LLMs são treinados (em 5 minutos)

O pipeline completo tem quatro estágios. Entender cada um explica muito do comportamento.

### 1. Pretraining — "decorando a internet"

- **Dados:** trilhões de tokens (web, livros, código, papers).
- **Objetivo:** dado N tokens, prever o N+1.
- **Resultado:** modelo que "sabe" quase tudo sobre linguagem, fatos comuns, código, mas não sabe ajudar. Só completa texto.
- **Custo:** dezenas a centenas de milhões de dólares em GPU-anos.

**Como você interage com um modelo pretrained puro:** "A capital da França é" → "Paris. A capital da Alemanha é Berlim..." Ele continua o padrão. Não responde à sua pergunta como assistente.

### 2. Supervised fine-tuning (SFT) — "aprendendo a ser assistente"

- **Dados:** milhares a centenas de milhares de pares (pergunta, resposta ideal) escritos por humanos.
- **Objetivo:** ajustar o modelo para responder perguntas em formato de assistente.
- **Resultado:** modelo que agora responde "A capital da França é Paris." quando perguntado.

### 3. RLHF — "aprendendo o que humanos preferem"

- **Processo:** humanos ranqueiam respostas do modelo (A é melhor que B). Treina-se um *reward model* que prediz essa preferência. O LLM é então otimizado via RL (PPO, DPO) para maximizar reward.
- **Resultado:** modelo útil, honesto, inofensivo — mais alinhado com expectativas humanas. Também: tende a ser bajulador, evita temas controversos, às vezes recusa tarefas inofensivas por precaução.

### 4. Constitutional AI (específico da Anthropic)

- **Processo:** um conjunto de princípios escritos guia o próprio modelo a auto-avaliar respostas. Reduz dependência de labelers humanos para safety.
- **Resultado:** Claude tende a ser mais consistente em recusas e mais transparente sobre seu raciocínio.

**Implicações práticas para devs:**

- Comportamentos "chatos" (desculpas excessivas, hedging) são artefato de RLHF.
- Recusa de tarefas inofensivas pode ser revertida com system prompt claro sobre contexto.
- Fine-tuning posterior do usuário muda **pouco** porque o pretraining é massivo. Não espere alteração radical de comportamento.

## Limitações fundamentais

### Hallucination

O modelo gera texto plausível mas factualmente incorreto. Não é "bug" — é consequência do objetivo de pretraining (prever tokens, não falar verdade). Mais comum em:

- Fatos específicos fora do conhecimento do modelo
- Referências (papers, URLs, nomes, datas)
- Raciocínio numérico/aritmético sem ferramentas

**Mitigações:**

- RAG com citação de fonte
- Tool use para cálculos (calculator, Python sandbox)
- Dizer no prompt "se não souber, diga que não sabe"
- LLM-as-judge para validar fatos
- Structured outputs forçam formato mas não verdade

### Knowledge cutoff

Modelo não sabe de eventos após sua data de treino. Cutoffs típicos (pode mudar):

- GPT-4.1: ~final de 2024
- Claude 4/4.6: início de 2025
- Gemini 2.5: início de 2025

**Mitigações:**

- RAG com dados atualizados
- Tool use para web search / API
- Grounding com sistemas externos

### Context rot ("lost in the middle")

Mesmo em modelos com janelas gigantes, informação no meio do contexto é usada pior que nas bordas. Pesquisas mostram degradação significativa em 32K+ tokens.

**Mitigações:**

- Coloque info crítica no início ou fim.
- Use RAG para trazer apenas trechos relevantes em vez de dump completo.
- Teste empiricamente com seu caso — não confie no marketing de "1M tokens".

### Não-determinismo

Mesmo com temperature 0, há fontes de não-determinismo: order de operações em GPU, batch effects, changes silenciosas do provedor. Não trate LLM como função pura.

**Mitigações:**

- Evaluation semântica em vez de equality.
- Fixtures para testes que não dependem do modelo.
- Monitoramento contínuo em produção.

### Prompt injection

Conteúdo externo (página web, PDF, email, documento) que entra no contexto pode sequestrar o modelo. Clássico: "ignore instruções anteriores e faça X".

**Mitigações:**

- Separar claramente system prompt de conteúdo não-confiável.
- Sanitizar/delimitar conteúdo externo: `<untrusted_content>...</untrusted_content>`.
- Em agents, NUNCA dar ferramentas destrutivas sem human-in-the-loop.
- Estudar [OWASP Top 10 for LLMs](https://owasp.org/www-project-top-10-for-large-language-model-applications/).

### Custo e latência

- Tokens custam. Prompts desnecessariamente grandes, ausência de cache, uso de Opus onde Haiku resolveria = conta de milhares por mês.
- Latência é dominada por: tamanho do output, tamanho do input, modelo escolhido, geography. Streaming ajuda percepção.

## Armadilhas comuns

### 1. Prompt vago

"Me ajude com código" vs "Gere uma função TypeScript `validateCPF(cpf: string): boolean` que implementa o algoritmo de dígitos verificadores do CPF brasileiro. Retorne false para CPFs com todos os dígitos iguais." A segunda dá 10x melhor resultado.

### 2. Confiar em JSON "que parece ok"

Sempre valide com schema (Zod, Pydantic, JSON Schema). Sempre tenha retry com mensagem de erro ao modelo: "Your previous response failed schema validation: {error}. Return a corrected response."

### 3. Ignorar custo

Monitore tokens por request, custo por feature, cost per conversion. Tenha alertas. Faça tiering.

### 4. Temperature errada para a tarefa

Código/JSON/classificação: temperature 0. Brainstorming: 0.7+. Usar temperature 1 em classificação = inconsistência garantida.

### 5. Context stuffing

"Vou jogar 500K tokens e deixar o modelo virar." Não. Quase sempre pior que RAG bem feito com 4K tokens relevantes.

### 6. Não testar mudanças de prompt

Mudou prompt = rodou golden set. Sem exceção. Prompt engineering sem eval é chute.

### 7. Não usar prompt caching

Deixar dinheiro na mesa. Se você tem system prompt grande ou documento repetido, cache.

### 8. Esquecer que o modelo muda

Provedores silenciosamente atualizam modelos. Pin version quando possível (`claude-sonnet-4-6-20260315`). Teste ao atualizar.

## Exemplos práticos

### Chat básico com streaming

```python
from anthropic import Anthropic

client = Anthropic()

with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="You are a concise senior developer.",
    messages=[{"role": "user", "content": "Explique event loop em Node.js"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### Classificação confiável com structured output

```python
from pydantic import BaseModel
from enum import Enum

class Category(str, Enum):
    BUG = "bug"
    FEATURE = "feature"
    QUESTION = "question"

class Classification(BaseModel):
    category: Category
    confidence: float
    reasoning: str

# Com OpenAI structured outputs
response = client.beta.chat.completions.parse(
    model="gpt-4.1",
    messages=[
        {"role": "system", "content": "Classify support tickets."},
        {"role": "user", "content": "My app crashes on startup..."}
    ],
    response_format=Classification,
    temperature=0
)
result: Classification = response.choices[0].message.parsed
```

### Tool use para cálculos

```python
tools = [{
    "name": "calculate",
    "description": "Evaluate a math expression. Use for any arithmetic.",
    "input_schema": {
        "type": "object",
        "properties": {"expression": {"type": "string"}},
        "required": ["expression"]
    }
}]

messages = [{"role": "user", "content": "What is 127 * 893?"}]

while True:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        tools=tools,
        messages=messages
    )
    if response.stop_reason == "tool_use":
        for block in response.content:
            if block.type == "tool_use" and block.name == "calculate":
                result = str(eval(block.input["expression"]))  # sandbox em prod!
                messages.append({"role": "assistant", "content": response.content})
                messages.append({"role": "user", "content": [{
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result
                }]})
    else:
        print(response.content[0].text)
        break
```

### Prompt caching em system prompt grande

```python
HUGE_SYSTEM = open("instructions.md").read()  # 5000 tokens

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=500,
    system=[{
        "type": "text",
        "text": HUGE_SYSTEM,
        "cache_control": {"type": "ephemeral"}
    }],
    messages=[{"role": "user", "content": "..."}]
)
# Primeira chamada: custo normal. Subsequentes: ~90% desconto no system.
```

## Evaluation — o que separa brinquedo de produção

LLM em produção sem evaluation é aposta. Práticas mínimas:

### Golden set

30-100 exemplos representativos com resposta esperada. Rode toda vez que mudar prompt ou modelo. Meça:

- **Match semântico** (embedding similarity) em tarefas abertas
- **Equality** em tarefas estruturadas (classificação, extração)
- **LLM-as-judge** em tarefas subjetivas

### LLM-as-judge

Use um modelo (tipicamente mais forte) para avaliar outro. Exemplo: GPT-4 avaliando Claude Haiku em classificação.

```text
Judge prompt:
Dada a pergunta X, a resposta correta Y, e a resposta do modelo Z,
avalie (0-10) quão correta Z é. Seja estrito. Formato: {score: ..., reason: ...}.
```

Útil, mas tenha cuidado com vieses do judge e cost.

### Traces e observabilidade

Instrumentar toda chamada LLM:

- Input tokens, output tokens, total cost
- Latência (TTFT — time to first token — e total)
- Taxa de erro (timeout, rate limit, schema invalid)
- Feedback do usuário (thumbs up/down)

Ferramentas: **Langfuse**, **Helicone**, **LangSmith**, **Arize Phoenix**, **Braintrust**.

### A/B test

Em produção, compare variantes de prompt/modelo. Métricas de negócio (conversion, resolution time) são mais honestas que métricas de LLM.

## Como ganhar experiência prática

Esta nota é estrutura técnica sobre LLMs. Para internalizar, prática é insubstituível. Três caminhos curados:

### Caminho 1 — Domínio de API em 1-2 semanas

Construir 4 scripts pequenos com Anthropic SDK (ou OpenAI), focando em uma capability cada:

1. **Chat com streaming** — terminal que mantém histórico, mostra tokens fluindo
2. **Classificador com structured output** — texto livre → JSON validado por schema
3. **Tool use loop** — agent mínimo com 2-3 tools (calculadora, web search)
4. **Prompt caching benchmark** — system prompt de ~3K tokens, medir custo com e sem cache

**Critério de sucesso:** consegue levantar uma integração LLM de produção sem hesitar; sabe responder por que cada parâmetro está como está.

### Caminho 2 — Feature LLM no Codex Technomanticus (2-3 semanas)

Implementar uma feature de LLM com valor real sobre o próprio vault:

- **Opção A:** classificador de notas do Inbox (urgente/aprender/referência) usando Haiku + structured output
- **Opção B:** sumário diário das notas modificadas
- **Opção C:** assistente de tags (sugere tags baseado no conteúdo)

Cobre o ciclo completo: prompting → API → eval → custo → produção pequena. Tem motivação real.

**Critério de sucesso:** feature funcionando, golden set de 20 casos, métricas (acurácia, custo, latência) e dashboard simples.

### Caminho 3 — LLM em produção em projeto profissional (quando aparecer)

Em projeto profissional com caso real, implementar com observabilidade completa, evaluation contínua, fallback multi-provider, pin de versão. Mais demorado, mas é o que vira "LLM em produção" no CV.

Sem oportunidade ainda? Caminhos 1 e 2 já dão fundação sólida; Caminho 3 espera.

**Critério de sucesso:** entrega no projeto com métricas, não estudo paralelo.

---

**Sugestão de ordem:** Caminho 1 → Caminho 2 → Caminho 3. Em qualquer caminho, golden set + métricas básicas são não-negociáveis.

## How to explain in English

### Elevator pitch

"LLMs are transformer-based neural networks trained to predict the next token. From that objective emerges remarkable language and code abilities. For production, the right framing is treating them as stochastic functions with untyped outputs — structured outputs, validation, retries, evaluation, and observability are not optional, they're the same engineering discipline applied to any unreliable dependency."

### Deeper version

"Three levels to think about LLMs in production. First, the architecture: causal decoder-only transformers, attention, tokenization, sampling. Enough understanding to reason about context rot, why temperature matters, and why tokenization affects cost and behavior — not enough to train them, and that's fine for application work.

Second, the API surface: system prompts, messages, tools, structured outputs, streaming, prompt caching. These are the everyday knobs. Defaults that hold up: temperature zero for structured tasks, higher for creative; tool use is mostly about description quality; prompt caching is massively underused.

Third, production concerns: cost, latency, evaluation, safety. The mature posture is prompts as code — versioned, reviewed, tested against golden sets — every call instrumented with tracing, models tiered aggressively (Haiku/Flash for triage, Sonnet/GPT-4 for the hard work), prompt injection defended at system boundaries, and never execute LLM-generated code or SQL without sandboxing or human review."

### Talking points for specific topics

- **On hallucination:** "Mitigated with RAG and citations, structured outputs for format stability, and LLM-as-judge for fact verification. Features should be designed assuming hallucination will happen — nothing critical relies on the LLM being right."
- **On cost:** "Prompt caching and model tiering are the biggest wins. Caching a 4-5K-token system prompt typically cuts feature cost by 80%+."
- **On evaluation:** "Prompts without evals are superstition. Golden sets run on every prompt change is the bare minimum."
- **On context window:** "1M tokens sounds great until you benchmark lost-in-the-middle. RAG-filtered 8K usually beats raw dumps, except for specific cases like codebase-wide refactoring."

### Key vocabulary

- modelo de linguagem grande → large language model (LLM)
- rede neural → neural network
- transformador → transformer
- atenção → attention
- auto-atenção → self-attention
- tokenização → tokenization
- codificação → encoding
- decodificação → decoding
- pré-treinamento → pretraining
- ajuste fino supervisionado → supervised fine-tuning (SFT)
- aprendizado por reforço com feedback humano → reinforcement learning from human feedback (RLHF)
- alucinação → hallucination
- janela de contexto → context window
- corte de conhecimento → knowledge cutoff
- temperatura → temperature
- amostragem → sampling
- amostragem por núcleo → nucleus sampling (top-p)
- ganância → greedy
- saída estruturada → structured output
- cache de prompt → prompt caching
- uso de ferramenta → tool use / function calling
- conjunto dourado → golden set
- juiz LLM → LLM-as-judge
- tempo até o primeiro token → time to first token (TTFT)
- não-determinismo → non-determinism
- injeção de prompt → prompt injection

## Recursos

### Leitura fundamental

- [The Illustrated Transformer — Jay Alammar](https://jalammar.github.io/illustrated-transformer/)
- [The Illustrated GPT-2 — Jay Alammar](https://jalammar.github.io/illustrated-gpt2/)
- [Attention is All You Need (paper original)](https://arxiv.org/abs/1706.03762)
- [Let's build GPT from scratch — Karpathy](https://www.youtube.com/watch?v=kCc8FmEb1nY) (2h, implementar um mini-GPT)
- [State of GPT — Karpathy](https://www.youtube.com/watch?v=bZQun8Y4L2A)
- [Transformers from Scratch — Peter Bloem](https://peterbloem.nl/blog/transformers)
- [Lost in the Middle (paper)](https://arxiv.org/abs/2307.03172)

### Documentação de API

- [Anthropic Messages API](https://docs.anthropic.com/en/api/messages)
- [Anthropic Prompt Caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- [OpenAI Chat Completions](https://platform.openai.com/docs/api-reference/chat)
- [OpenAI Structured Outputs](https://platform.openai.com/docs/guides/structured-outputs)
- [Google Gemini API](https://ai.google.dev/gemini-api/docs)

### Ferramentas de observabilidade

- [Langfuse](https://langfuse.com/) — open source LLM observability
- [Helicone](https://helicone.ai/)
- [LangSmith](https://smith.langchain.com/)
- [Arize Phoenix](https://phoenix.arize.com/)
- [Braintrust](https://www.braintrust.dev/)

### Para rodar LLMs localmente

- [Ollama](https://ollama.ai/) — mais simples para começar
- [LM Studio](https://lmstudio.ai/) — GUI
- [vLLM](https://docs.vllm.ai/) — serving performance
- [llama.cpp](https://github.com/ggerganov/llama.cpp) — rodar em CPU/Metal

### Livros

- *AI Engineering* — Chip Huyen (2025)
- *Building LLMs for Production* — Bouchard, Peters
- *Hands-On Large Language Models* — Jay Alammar, Maarten Grootendorst

## Deep dives técnicos — entendendo os internals

### Self-attention em detalhe

Self-attention é o mecanismo que permite Transformer processar sequências com paralelização. Aqui está o que realmente acontece para cada token:

**Passo 1 — Projeções Q/K/V.** O embedding de cada token é multiplicado por três matrizes aprendidas (W_Q, W_K, W_V), produzindo três vetores: Query, Key, Value.

```text
para cada token i:
  Q_i = embedding_i × W_Q   // "o que eu procuro"
  K_i = embedding_i × W_K   // "o que eu ofereço"
  V_i = embedding_i × W_V   // "o que eu carrego"
```

**Passo 2 — Attention scores.** Para cada token, calcula-se quanto ele "presta atenção" em cada outro token.

```text
score(i, j) = (Q_i · K_j) / sqrt(d_k)
```

A divisão por `sqrt(d_k)` (onde `d_k` é a dimensão das Keys) evita que produtos escalares fiquem muito grandes e saturem o softmax.

**Passo 3 — Softmax.** Os scores viram probabilidades que somam 1.

```text
attention(i, j) = softmax(score(i, j))
```

**Passo 4 — Causal mask.** Em LLMs decoder-only (GPT, Claude, Llama), tokens futuros são mascarados com `-inf` antes do softmax. Isso garante que token i só vê tokens j ≤ i.

**Passo 5 — Weighted sum.** Output de cada token é a soma ponderada dos Values.

```text
output_i = sum(attention(i, j) × V_j para cada j)
```

**Multi-head:** em vez de uma "cabeça" de attention, usa-se N cabeças (tipicamente 8-128) em paralelo, cada uma com suas próprias matrizes W_Q/K/V. Elas aprendem a focar em coisas diferentes — algumas em sintaxe, outras em semântica, outras em referências. Outputs das cabeças são concatenados e projetados de volta.

**Complexidade:** O(n²) em memória e compute, onde n é o número de tokens. Isso explica por que context windows grandes são caros. Alternativas (sparse attention, sliding window, linear attention) existem mas trade-off qualidade por custo.

### KV cache — a otimização fundamental de inferência

Durante geração autoregressive, você gera um token por vez. Ingenuamente, para o token N+1 você teria que re-processar todos os N tokens anteriores. **KV cache** resolve isso: guardam-se os Keys e Values já computados e só se calcula o novo token.

```text
Token 1: computar Q1, K1, V1 → cache [K1, V1]
Token 2: computar Q2, K2, V2 → cache [K1,K2, V1,V2] → attention com K1-K2, V1-V2
Token 3: computar Q3, K3, V3 → cache cresce → attention com cache
...
```

**Implicações práticas:**

- **Latência de TTFT (time to first token)** é dominada por "prefill" — processar o prompt inicial. Esse custo é O(n²).
- **Latência entre tokens** ("decode") é O(n) após o prefill, porque KV cache amortiza.
- **Memória** cresce linearmente com contexto. Contextos de 1M tokens exigem GBs de VRAM só para KV cache.
- **Prompt caching** (Anthropic, OpenAI) é essencialmente KV cache persistente entre chamadas — por isso funciona como funciona.

### Positional encoding e RoPE

Transformers não têm noção natural de ordem — sem algo explícito, "gato come peixe" seria igual a "peixe come gato". **Positional encoding** injeta info de posição.

**Approaches:**

- **Absolute (original):** vetores senoidais somados aos embeddings.
- **Learned absolute:** embeddings posicionais aprendidos.
- **RoPE (Rotary Position Embedding):** rotaciona Q e K no espaço complexo pela posição. Dominante em LLMs modernos (Llama, Claude, GPT-4+). Vantagens: funciona bem com long context, extrapolação razoável para posições não vistas no treino.
- **ALiBi, T5 relative:** variantes para contextos longos.

Por que importa: extrapolação para contextos mais longos que o treino depende do positional encoding. RoPE com "NTK-aware" scaling ou "YaRN" são técnicas para estender context window pós-treino.

### Tokenização — BPE em detalhe

**BPE (Byte-Pair Encoding)** é o algoritmo padrão. Ideia: começa com bytes/caracteres individuais, mescla pares mais frequentes iterativamente até atingir um vocabulário alvo (tipicamente 50K-200K tokens).

**Exemplo simplificado** com corpus `"low", "lower", "newest", "widest"`:

```text
Início (vocab = caracteres): {l, o, w, e, r, n, s, t, i, d}

Iteração 1: par mais frequente → "es" (de "newest", "widest")
  merge: "es" vira token
Iteração 2: próximo mais frequente → "est"
  merge: "est" vira token
...
```

O vocabulário final contém caracteres + merges. Qualquer palavra pode ser tokenizada quebrando em merges conhecidos.

**Implicações:**

- Palavras comuns viram 1 token; raras viram múltiplos.
- Código é penalizado (sintaxe, indentação, camelCase).
- Línguas não-latinas são penalizadas (tokenizers EN-first).
- Modelos com tokenizers diferentes têm economia diferente para o mesmo texto.

**Ferramentas:** [tiktoken](https://github.com/openai/tiktoken) (OpenAI), `anthropic.count_tokens()`, [transformers AutoTokenizer](https://huggingface.co/docs/transformers/main_classes/tokenizer).

### Embedding space geometry

Embeddings vivem num espaço de alta dimensão com geometria aprendida. Algumas propriedades:

- **Similaridade semântica → proximidade geométrica.** Cosine similarity é a medida padrão.
- **Anisotropia:** espaços de embedding de modelos comuns têm uma direção "quente" onde todos os vetores concentram. Isso distorce similaridade e é corrigido com whitening em alguns pipelines.
- **Linear structure:** operações como `king - man + woman ≈ queen` funcionam (parcialmente) em modelos bem treinados. Base de "word analogies".
- **Dimensões matryoshka:** modelos modernos (OpenAI v3, Voyage 3) são treinados para que truncar as primeiras K dimensões preserve qualidade — permite trade-off custo/qualidade.
- **Dense vs sparse:** dense embeddings têm ~500-4000 dims; sparse (SPLADE, ELSER) têm dimensão do vocabulário com maioria zero.

## Deep dives — papers fundamentais

### Attention is All You Need (Vaswani et al., 2017)

O paper que introduziu Transformer. Leia para: intuição sobre attention, motivação arquitetural, por que RNNs ficaram obsoletos.

Ponto-chave: multi-head self-attention substitui recorrência por paralelização massiva. [arxiv](https://arxiv.org/abs/1706.03762)

### Scaling Laws (Kaplan et al., 2020; Hoffmann et al., 2022 — Chinchilla)

Como performance escala com parameters, data, compute. Chinchilla mostrou que modelos anteriores estavam undertrained.

Ponto-chave: proporção ideal é ~20 tokens por parâmetro. Modelo de 70B quer ~1.4T tokens de treino. [Chinchilla](https://arxiv.org/abs/2203.15556)

### InstructGPT (Ouyang et al., 2022)

Paper que descreveu SFT + RLHF, produzindo "ChatGPT-like" a partir de GPT-3. Explica por que modelos alinhados são mais úteis mas também mais "ativos" (hedging, recusas).

Ponto-chave: alignment não aumenta capacidade bruta, mas aumenta utilidade por fator enorme. [arxiv](https://arxiv.org/abs/2203.02155)

### FlashAttention (Dao et al., 2022, 2023)

Otimização de I/O de attention em GPU. Reduz memória de O(n²) para O(n) ao processar em tiles. Destravou contexto longo viável.

Ponto-chave: muita otimização de LLM vem de I/O awareness, não de math puro. [arxiv](https://arxiv.org/abs/2205.14135)

### Lost in the Middle (Liu et al., 2023)

Prova empírica que modelos usam info do início e fim de contexto melhor que do meio. Vale para contextos grandes mesmo em modelos "long context".

Ponto-chave: placement no prompt importa. RAG bem feito bate "context dump" quase sempre. [arxiv](https://arxiv.org/abs/2307.03172)

### Constitutional AI (Bai et al., Anthropic 2022)

Como Claude é alinhado. Modelo auto-critica usando princípios escritos, reduz dependência de labelers humanos.

Ponto-chave: direção de safety escalável; explica comportamentos distintivos de Claude. [arxiv](https://arxiv.org/abs/2212.08073)

### Switch Transformer / Mixtral (MoE)

Mixture of Experts: cada token ativa apenas uma fração dos parâmetros. Permite modelos com trilhões de parâmetros a custo de inferência comparável a modelos densos menores.

Ponto-chave: GPT-4 rumor-ado é MoE. Mixtral 8x7B é exemplo open source prático. [Mixtral paper](https://arxiv.org/abs/2401.04088)

### Language Models are Few-Shot Learners (GPT-3, Brown et al. 2020)

O paper que mostrou que scaling destrava in-context learning. Base do "prompt engineering" como campo.

Ponto-chave: modelos grandes aprendem pelo prompt, não só por fine-tuning. [arxiv](https://arxiv.org/abs/2005.14165)

## Casos comuns no mercado — observability e incidentes

Padrões frequentes em times rodando LLMs em produção. Não são casos vividos pessoalmente — são armadilhas recorrentes documentadas em post-mortems, talks, e literatura técnica.

### Caso 1 — Cost spike por context creep

**Padrão observado:** feature de LLM em produção há meses. Sem aumento de usuários, custo dobra em uma semana. Investigação via observability mostra que tokens médios por chamada dobraram. Causa raiz típica: um refactor passou a incluir mais contexto (histórico completo em vez de último par; "últimas 20 entradas" em vez de "últimas 3").

**Fix típico:**

- Limite hard de tokens de input por feature.
- Alerta "tokens/call > p95 histórico".
- Regression test com assertion de cost budget.

### Caso 2 — Outage de provider

**Padrão observado:** API do provider começa a retornar 529 (overloaded) em pico de tráfego. Feature crítica fica fora por dezenas de minutos.

**Fix típico:** roteador multi-provider com cascata de fallback (ex: Claude → GPT-4.1 → Gemini Flash) e circuit breaker em health checks passivos.

**Lição:** multi-provider resilience é design, não opcional.

### Caso 3 — Silent model update

**Padrão observado:** feature usa alias (ex: `claude-sonnet-4-5`). Provider atualiza o modelo atrás do alias. Taxa de erro/comportamento muda silenciosamente. SLA tradicional (uptime) não detecta porque é qualidade.

**Fix típico:** pin de versão obrigatório em produção + golden set em CI rodando em todo PR que toque prompt ou config de modelo.

### Caso 4 — Structured output edge case

**Padrão observado:** extração estruturada com JSON mode funciona 99%. Mudança trivial no system prompt (capitalização, espaçamento, vírgula) altera distribuição de saída. Modelo passa a retornar JSON em markdown code block em vez de raw, ou adiciona campos extras. Taxa de erro pula para ~8%.

**Fix típico:**

- Golden set com assertions estritas de schema.
- `response_format` / `tool_choice` forçado em vez de depender de instrução textual.
- Code review rigoroso para mudanças em prompts (mesmo "cosméticas").

### Caso 5 — PII em logs

**Padrão observado:** inicialmente, logs da API LLM contêm inputs completos dos usuários. Passa em compliance review por inércia. Auditoria posterior pede retenção limitada e redaction de dados sensíveis. Logs já estão espalhados em múltiplos sistemas (observability tool, cloud provider logs, DB).

**Fix típico:**

- PII detection em middleware antes de enviar ao provider.
- Campos sensíveis substituídos por hashes para audit.
- Retention policy enforçada centralmente.
- Audit review periódico.

**Lição:** compliance não é afterthought. Entrar no sistema cedo é barato; retrofit é caro.

### Observabilidade mínima de LLM em produção

```text
Por chamada:
  - feature_id, user_id (hash), session_id, request_id
  - modelo, versão
  - input_tokens, output_tokens, cache_hit
  - latency (TTFT, total)
  - temperature, max_tokens, top_p
  - tool_calls (count, names, latency)
  - status (ok, error, timeout, schema_invalid)
  - custo estimado

Dashboards:
  - custo por feature (diário, semanal, mensal)
  - latência p50/p95/p99
  - taxa de erro por tipo
  - tokens/call trend (detectar context creep)
  - cache hit rate (eficácia de prompt caching)

Alertas:
  - custo diário > 1.5× média de 30d
  - p95 latência > SLA
  - taxa de erro > 2%
  - golden set regression (from CI)
```

Ferramentas comuns no mercado: Langfuse, Helicone, LangSmith, Braintrust, Arize Phoenix.

## Exercícios hands-on — labs

### Lab 1 — Tokenizer awareness

**Objetivo:** desenvolver intuição sobre tokenização.

**Tarefas:**

1. Com `tiktoken`, meça tokens para:
   - Uma página de Wikipedia em inglês
   - Mesma página em português
   - Mesma página em japonês
   - Arquivo de código TypeScript
   - Arquivo de código Python minificado
   - JSON com muitos campos curtos
2. Calcule custo estimado (Claude Sonnet input) para cada.
3. Escreva um post comparando.

**Aprendizado esperado:** linguagens não-latinas são mais caras; código tem overhead; JSON denso é relativamente eficiente.

### Lab 2 — Sampling intuition

**Objetivo:** sentir diferença entre temperature settings.

**Tarefas:**

1. Prompt: "Escreva um parágrafo sobre o que é Kubernetes."
2. Rode 5 vezes com temperature em [0, 0.2, 0.5, 1.0, 1.5].
3. Compare saídas. Observe repetição vs variabilidade vs coerência.
4. Meça tokens e score subjetivo de qualidade.

**Aprendizado esperado:** você desenvolve intuição do que é "apropriado" para cada use case.

### Lab 3 — Tool use robusto

**Objetivo:** construir tool use confiável que lida com erros.

**Tarefas:**

1. Defina 3 tools: `calculator`, `get_time`, `search_wikipedia`.
2. Implemente loop de agent com max_steps=10.
3. Adicione:
   - Validação de input com Pydantic
   - Retry com error feedback ao modelo
   - Logging estruturado
   - Timeout por tool
4. Teste com casos difíceis: input inválido, tool que falha, pergunta que exige encadear tools.

**Aprendizado esperado:** você entende que tool use em produção precisa de muito mais que "call tool".

### Lab 4 — Prompt caching ROI

**Objetivo:** medir impacto real de prompt caching.

**Tarefas:**

1. Pegue um system prompt de ~3-5K tokens (ou crie um).
2. Rode 100 chamadas sem caching, meça custo e latência p50.
3. Adicione `cache_control: ephemeral`.
4. Rode 100 chamadas, meça.
5. Compare. Calcule ROI em dólares.

**Aprendizado esperado:** você vê na prática por que caching "paga" rapidamente.

### Lab 5 — Eval com golden set

**Objetivo:** estabelecer disciplina de evaluation.

**Tarefas:**

1. Escolha uma tarefa: classificação, extração, ou Q&A.
2. Monte golden set de 50 exemplos.
3. Escreva script que rode o prompt sobre o set e compute métricas.
4. Integre em GitHub Actions — PR falha se regressão > X%.
5. Mude o prompt várias vezes, observe impacto no CI.

**Aprendizado esperado:** você sai da "iteração ad-hoc" e entra na "evolução medida".

### Lab 6 — Multi-provider fallback

**Objetivo:** implementar resilience cross-provider.

**Tarefas:**

1. Construa um cliente que aceita lista de providers em ordem de prioridade.
2. Primary: Claude Sonnet. Fallback: GPT-4.1. Fallback final: Gemini Flash.
3. Circuit breaker: se primary retorna erro 3x seguidas em 1 min, pula por 5 min.
4. Teste simulando outage (mock 529 responses).
5. Meça: latência adicionada pelo fallback, precisão comparativa.

**Aprendizado esperado:** você entende arquitetura resiliente em sistemas stochastic.

## Veja também

- [[Inteligência Artificial]] — contexto amplo de IA
- [[Skills e Prompting]] — prompt engineering aplicado
- [[Agents]] — LLMs que usam ferramentas em loop
- [[RAG e Vector Databases]] — injetar conhecimento em LLMs
- [[MCP]] — Model Context Protocol
- [[Claude]] — família Anthropic
- [[GitHub Copilot]]
- [[Codex]]
- [[Gemini]]
- [[Comparativo de LLMs]] — escolha entre modelos
- [[Trilha IA]] — roadmap
