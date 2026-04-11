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

**Impacto real:** no MedEspecialista reduzimos custo de sumarização em ~85% movendo o prompt-padrão (~4K tokens) para cache. A latência também caiu ~40%.

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

## Na prática (da minha experiência)

No MedEspecialista, minha experiência operacional com LLMs foi uma progressão dolorosa que destravou padrões:

**Lição 1 — Prompt caching paga sozinho a feature.** Feature de sumarização estava custando ~$2.400/mês em Sonnet. Movi system prompt e few-shot examples para cache, custo caiu para ~$350/mês. ROI imediato.

**Lição 2 — Tiering é obrigatório em escala.** Para triagem de mensagens (classificar urgente/normal/junk), Opus era overkill. Haiku com 8 few-shot examples cobre 95% dos casos a 1/30 do custo. Reservo Opus para os 5% que o Haiku marca como "incerto".

**Lição 3 — Structured outputs eliminam classes inteiras de bugs.** Antes de usar tool use forçado com schema, 2-3% dos responses vinham com JSON quase-válido (trailing comma, aspas erradas, campo extra). Todo parse era envelope com retry. Depois de structured outputs, essa taxa caiu para praticamente zero.

**Lição 4 — Context rot é real.** Tentei passar o prontuário completo (120K tokens) para o modelo sumarizar. Resultado: perdia info do meio. Mudei para pipeline de chunking + summary-of-summaries. Qualidade subiu, custo caiu.

**Lição 5 — Observabilidade salva seu pescoço.** Integrei Langfuse em todos os calls. Um dia, custo dobrou do nada. Olhei no dashboard: um dev tinha colocado um PDF de 80 páginas no prompt dentro de um loop. Detectei em 2h, não em final de mês com susto.

**Lição 6 — Pin model versions em produção.** Dia que Anthropic atualizou Sonnet silenciosamente, um prompt que era 99% estável começou a falhar em ~15% dos casos. Versões pinadas em config + suite de regression tests passou a ser obrigatório.

**Incidente memorável:** um refactor de prompt "aparentemente inofensivo" quebrou a extração de dados estruturados — adição de um tab no system prompt fez o modelo começar a retornar código markdown em vez de JSON puro. Passou no code review, passou em 3 testes manuais, quebrou em produção quando hit alguns inputs específicos. Desde então: golden set de 50 casos, rodado automaticamente em todo PR que toca prompt.

## How to explain in English

### Elevator pitch

"LLMs are transformer-based neural networks trained to predict the next token. The magic is that from that objective emerges remarkable language and code abilities. For production work, my focus is on treating them as stochastic functions with untyped outputs — I use structured outputs, validation, retries, evaluation, and observability the same way I'd engineer any other unreliable dependency."

### Deeper version

"I think about LLMs at three levels. First, the architecture: causal decoder-only transformers, attention, tokenization, sampling. I don't train them, but I know enough to reason about context rot, why temperature matters, and why tokenization affects cost and behavior.

Second, the API surface: system prompts, messages, tools, structured outputs, streaming, prompt caching. These are the knobs I use every day. I have opinions about temperature (zero for structured tasks, higher for creative), about tool use (descriptions are everything), about caching (massively underused).

Third, production concerns: cost, latency, evaluation, safety. I treat prompts as code — versioned, reviewed, tested against golden sets. I instrument every call with tracing. I tier models aggressively — Haiku for triage, Sonnet for the hard work. I defend against prompt injection at system boundaries. And I never, ever execute LLM-generated code or SQL without sandboxing or human review."

### Talking points for specific topics

- **On hallucination:** "I mitigate it with RAG and citations, structured outputs for format stability, and LLM-as-judge for fact verification. And I design features assuming hallucination will happen — nothing critical relies on the LLM being right."
- **On cost:** "Prompt caching and model tiering are the biggest wins. I cut a feature's cost by 85% just by caching the system prompt."
- **On evaluation:** "Prompts without evals are superstition. I maintain golden sets and run them on every prompt change. That's the bare minimum."
- **On context window:** "1M tokens sounds great until you benchmark lost-in-the-middle. I still prefer RAG-filtered 8K over raw dumps, except for very specific use cases like codebase-wide refactoring."

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
