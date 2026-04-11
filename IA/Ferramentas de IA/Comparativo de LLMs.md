---
title: "Comparativo de LLMs"
created: 2026-04-01
updated: 2026-04-11
type: concept
status: evergreen
tags:
  - ia
  - llm
  - ferramentas
  - entrevista
publish: true
---

# Comparativo de LLMs

> "Qual LLM devo usar?" é a pergunta que todo senior fullstack responde 20 vezes por ano. Esta nota é o framework de decisão que uso hoje, destilado de ~3 anos pagando contas de API, testando migrações, e revertendo decisões mal informadas. **Não existe "o melhor LLM"** — existe o melhor para cada combinação de (task, restrições de custo, restrições de latência, stack, compliance). Esta nota dá: uma matriz de decisão prática, trade-offs reais (não marketing), e critérios para escolher entre Claude, GPT, Gemini e ferramentas derivadas em 2026. Para notas individuais, ver [[Claude]], [[GitHub Copilot]], [[Codex]], [[Gemini]]. Para fundamentos de LLMs em geral, [[LLMs]].

## A pergunta errada e a pergunta certa

**Pergunta errada:** "Qual o melhor LLM?"

**Pergunta certa:** "Dada minha task, meu budget, minha latência tolerável, meu stack e minhas restrições de compliance, qual LLM entrega o melhor resultado esperado?"

Sempre que alguém pergunta a primeira, eu pergunto de volta:

1. **Task:** coding interativo? Classificação em volume? Chatbot? RAG? Multimodal?
2. **Custo tolerável:** $0.001 por chamada ou $0.30?
3. **Latência:** tempo real (< 500ms) ou async (minutos)?
4. **Stack existente:** já usa GCP? AWS? GitHub? Nenhum?
5. **Compliance:** regulamentado (saúde, finance)? Data residency? On-prem?
6. **Volume:** 100 chamadas/dia ou 10M/dia?

Só depois dessas respostas faço sentido recomendar algo.

## As três famílias principais em 2026

| Família | Empresa | Modelos principais | Context max | Força definidora |
| --- | --- | --- | --- | --- |
| **Claude** | Anthropic | Opus 4.x, Sonnet 4.6, Haiku 4.5 | 1M tokens | Raciocínio profundo, código, tool use consistente, safety |
| **GPT** | OpenAI | GPT-4.1, GPT-4o, o1, o3 | 1M tokens (dependendo) | Ecossistema amplo, multimodal, voice, integração enterprise |
| **Gemini** | Google DeepMind | 2.5 Pro, 2.5 Flash, 2.5 Flash-Lite | 1M-2M tokens | Multimodal nativo, contexto gigante, integração Google Cloud |

Fora dessas, há players relevantes em nicho:

- **Llama 3/4 (Meta)** — open source state-of-the-art, base para self-hosting.
- **Mistral** — open source europeu, competitivo, compliance-friendly.
- **DeepSeek** — open source chinês, muito forte em coding, custo agressivo.
- **Qwen (Alibaba)** — open source, forte em multilingual (incluindo PT-BR).

## O que diferencia um senior escolhendo LLM

1. **Não decide por benchmark público.** Benchmarks decoram tasks; seu workload é único. Testa com golden set próprio.
2. **Separa modelo de ferramenta.** GPT-4.1 ≠ Copilot. Claude ≠ Claude Code. Ferramentas adicionam workflow, modelo é modelo.
3. **Tiering é default, não otimização.** Haiku/Flash para triagem, escalada para Sonnet/4.1, último recurso Opus/o3.
4. **Pin version em produção.** Nunca alias em críticos.
5. **Conhece os custos reais** — input tier, output tier, caching, batch.
6. **Testa migração antes de decidir.** Comparação side-by-side em amostra representativa.
7. **Considera total cost of ownership** — API + observabilidade + dev time + treinamento.
8. **Sabe onde cada modelo falha.** Claude pode ser cauteloso demais; GPT pode alucinar com confiança; Gemini pode ser inconsistente em text-only.
9. **Pensa em lock-in.** Migração cross-provider leva tempo; escolha com isso em mente.
10. **Revisa decisão a cada 6 meses.** Paisagem muda rápido.

## Matriz de decisão por task

### Coding interativo (pair programming)

**Ferramentas:** Claude Code, Cursor, Copilot Agent Mode, Gemini CLI.

| Critério | Recomendação |
| --- | --- |
| **Default** | **Claude Code** (Claude Opus/Sonnet) — raciocínio e tool use consistentes |
| **IDE-first, GitHub-centric** | **Copilot** (com Claude ou GPT como model) |
| **Codebase muito grande** | **Gemini CLI** ou Claude Code Opus (context longo) |
| **Async/paralelo** | **Codex Cloud** |
| **Voice/visual interaction** | **Cursor** ou **Gemini Live** (experimental) |

Minha stack atual: Claude Code + Copilot (inline completions). Os dois juntos batem qualquer um sozinho.

### Chatbot em produção

| Critério | Recomendação |
| --- | --- |
| **Qualidade de conversa** | Claude Sonnet 4.6 ou GPT-4.1 |
| **Voice** | GPT-4o Voice ou Gemini Live |
| **Custo baixo** | Gemini Flash ou Claude Haiku |
| **Custo extremo** | Gemini Flash-Lite ou self-host Llama 3 |
| **Multilingual incluindo PT-BR** | Claude Sonnet ou Qwen (self-host) |
| **Compliance health/finance** | Vertex AI + Gemini ou Azure OpenAI + GPT |

### Classificação / extração em volume

| Critério | Recomendação |
| --- | --- |
| **Default** | Claude Haiku ou GPT-4o-mini ou Gemini Flash-Lite |
| **Cost-king** | **Gemini Flash-Lite** — frequentemente 2-3x mais barato |
| **Qualidade premium** | Claude Sonnet com structured outputs |
| **Self-host** | Llama 3 70B ou DeepSeek (coding) |

### RAG e search

| Critério | Recomendação |
| --- | --- |
| **Generator** | Claude Sonnet (seguir contexto com fidelidade) |
| **Embeddings** | Voyage 3, Cohere embed v3, ou OpenAI text-embedding-3-large |
| **Reranker** | Cohere Rerank v3 ou Voyage Rerank |
| **Grounding web** | Gemini com Search grounding |

Ver [[RAG e Vector Databases]] para deep dive.

### Agents com tool use

| Critério | Recomendação |
| --- | --- |
| **Default** | Claude Sonnet — tool use mais consistente |
| **Raciocínio pesado** | Claude Opus ou GPT o3 |
| **Cloud sandbox** | Codex (OpenAI) |
| **Framework** | Claude Agent SDK, LangGraph, Vercel AI SDK |

### Multimodal

| Critério | Recomendação |
| --- | --- |
| **Imagem/OCR** | Gemini 2.5 Pro ou Claude Sonnet (vision) |
| **Vídeo** | **Gemini 2.5 Pro** (único com nativo robusto) |
| **Áudio entrada/saída** | GPT-4o ou Gemini Live |
| **Combinação de múltiplos modos** | **Gemini 2.5 Pro** |

### Raciocínio matemático/lógico

| Critério | Recomendação |
| --- | --- |
| **Top tier** | **OpenAI o1 / o3** ou Claude Opus com extended thinking |
| **Budget** | Claude Sonnet com CoT ou self-consistency |

### Casos regulados (saúde, finance, governo)

| Critério | Recomendação |
| --- | --- |
| **Azure-first** | Azure OpenAI (GPT) |
| **GCP-first** | Vertex AI (Gemini ou Claude via Vertex) |
| **AWS-first** | Bedrock (Claude, Llama, Mistral) |
| **On-prem obrigatório** | Llama 3, Mistral, Qwen self-host |

## Matriz técnica comparativa

### Capacidades

| Aspecto | Claude 4.x | GPT-4.1/o3 | Gemini 2.5 |
| --- | --- | --- | --- |
| **Context window** | 1M | 1M | 1M-2M |
| **Multimodal text+img** | Sim | Sim | Nativo (melhor) |
| **Multimodal video** | Não nativo | Não nativo | **Sim nativo** |
| **Multimodal audio** | Não | GPT-4o | Sim nativo |
| **Voice mode** | Não | GPT-4o Voice | Gemini Live |
| **Extended thinking** | Sim (transparente) | o1/o3 (hidden) | Em Pro |
| **Tool use nativo** | Muito bom | Bom | Bom |
| **Structured outputs** | Via tool use | Sim nativo | Sim |
| **Prompt caching** | Sim (~90% desconto) | Sim (50%) | Context caching |
| **Batch API** | Sim (50% desconto) | Sim | Sim |
| **Fine-tuning** | Não (em breve?) | Sim | Sim |
| **Grounding web** | Não nativo | Via Assistants API | **Google Search nativo** |

### Custo (2026, aproximado, por 1M tokens)

| Modelo | Input | Output | Caching |
| --- | --- | --- | --- |
| **Claude Opus 4.x** | $15 | $75 | -90% |
| **Claude Sonnet 4.6** | $3 | $15 | -90% |
| **Claude Haiku 4.5** | $0.80 | $4 | -90% |
| **GPT-4.1** | $3 | $12 | -50% |
| **GPT-4.1-mini / 4o-mini** | $0.15-0.25 | $0.60-1.00 | -50% |
| **o3** | $15+ | $60+ | -50% |
| **Gemini 2.5 Pro** | $1.25-2.50 | $5-10 | Sim |
| **Gemini 2.5 Flash** | $0.30 | $2.50 | Sim |
| **Gemini 2.5 Flash-Lite** | $0.10 | $0.40 | Sim |

**Observação:** tiers de contexto longo, batch, caching e regional pricing podem fazer diferença enorme. Sempre consulte pricing atual do provedor.

### Qualidade de código (observação prática)

Baseado em minha experiência e benchmarks do meu golden set pessoal:

- **Refactors complexos, debugging profundo:** Claude Opus ~ o3 > Claude Sonnet > GPT-4.1 > Gemini 2.5 Pro
- **Coding interativo confortável:** Claude Code > Cursor > Copilot > Gemini CLI
- **Completions inline rápidas:** Copilot ~ Cursor > Claude Code
- **Tool use confiabilidade:** Claude > GPT > Gemini
- **Seguir instruções longas:** Claude > GPT > Gemini

### Segurança e safety

- **Claude:** Constitutional AI, mais conservador, admite "não sei" com mais frequência.
- **GPT:** moderação maduras, mas ocasionalmente alucina com confiança.
- **Gemini:** moderação forte, ocasionalmente bloqueia tasks legítimas.

Para tasks sensíveis: sempre prompt injection defense + output filtering + human-in-the-loop em ações destrutivas.

## Ferramentas derivadas — comparativo

### Coding agents locais/cloud

| Ferramenta | Modelo base | Tipo | Pontos fortes | Pontos fracos |
| --- | --- | --- | --- | --- |
| **Claude Code** | Claude Opus/Sonnet | CLI + IDE interativo | Raciocínio, tool use, skills, MCP, CLAUDE.md | CLI learning curve |
| **Cursor** | Múltiplos (GPT, Claude, custom) | IDE AI-first | UX excelente, tab completion, composer | Lock-in no IDE |
| **GitHub Copilot** | GPT + Claude + Gemini | IDE inline + chat + agent | Integração GitHub, multi-IDE, PR automation | Menos customizável |
| **Codex (OpenAI)** | GPT-4.1, o1, o3 | Agent cloud async | Paralelização, PR automation | Não interativo |
| **Gemini CLI** | Gemini 2.5 | CLI | Multimodal, grounding, contexto 2M | Ecosistema menor |
| **Gemini Code Assist** | Gemini | IDE extension | Integração GCP | Menos features que Copilot |
| **Aider** | Múltiplos (user escolhe) | CLI | Open source, controle fino | Requer config |
| **Cline** | Múltiplos | VS Code extension | Open source, flexível | Menos polido que Claude Code |
| **Continue** | Múltiplos | VS Code/JetBrains | Open source, multi-model | Maturidade variável |

### Frameworks de agent

| Framework | Provider | Pontos fortes | Pontos fracos |
| --- | --- | --- | --- |
| **Claude Agent SDK** | Anthropic | Nativo, MCP, observabilidade | Claude-first |
| **LangChain/LangGraph** | Community | Ecossistema gigante | Overhead, breaking changes |
| **Vercel AI SDK** | Vercel | Next.js/React DX excelente | Focado em web |
| **CrewAI** | Community | Multi-agent paradigm claro | Menos maduro |
| **AutoGen** | Microsoft | Academic rigor | Menos prod-ready |
| **llamaindex** | Community | RAG-focused | Menos agent-focused |

### Plataformas enterprise

| Plataforma | Provider | Pontos fortes |
| --- | --- | --- |
| **Azure OpenAI** | Microsoft + OpenAI | GPT com enterprise guarantees, Office integration |
| **AWS Bedrock** | AWS | Multi-model (Claude, Llama, Mistral), IAM, region selection |
| **Google Vertex AI** | Google | Gemini + Claude + modelos, governance, model garden |
| **Anthropic Console** | Anthropic | Claude direto, mais simples |
| **OpenAI Platform** | OpenAI | GPT direto, Assistants API |

## Framework de decisão prático

Quando você tem que escolher **agora**, use este fluxograma mental:

```text
1. É tarefa multimodal (imagem, vídeo, áudio)?
   ├── Sim → Gemini 2.5 Pro (ou GPT-4o para voice)
   └── Não → continua

2. É coding interativo pair programming?
   ├── Sim → Claude Code (Claude Sonnet/Opus)
   └── Não → continua

3. É classificação/extração em volume (> 100K/dia)?
   ├── Sim → Gemini Flash-Lite, Claude Haiku, GPT-4o-mini (benchmark)
   └── Não → continua

4. Precisa de raciocínio matemático/lógico pesado?
   ├── Sim → OpenAI o3 ou Claude Opus com extended thinking
   └── Não → continua

5. É chatbot/assistant de produção?
   ├── Sim → Claude Sonnet 4.6 como default, Haiku para simples
   └── Não → continua

6. Compliance: saúde, finance, gov?
   ├── Sim → Azure OpenAI / Vertex AI / Bedrock com contratos
   └── Não → continua

7. Default: Claude Sonnet 4.6 via Anthropic API.
```

Se seu projeto tem múltiplas necessidades, tier agressivamente — não escolha um modelo para tudo.

## Padrões de arquitetura

### Padrão 1: tiering agressivo

```text
User request
  │
  ▼
Haiku/Flash-Lite triagem
  │
  ├── "Simples" → responde direto
  │
  ├── "Média" → Sonnet/GPT-4.1/Gemini Flash
  │
  └── "Complexa" → Opus/o3/Gemini Pro
```

Redução típica de custo: 5-10x vs usar modelo grande sempre.

### Padrão 2: multi-provider fallback

```text
Primary: Claude Sonnet (quality)
  │
  ├── on error/timeout → GPT-4.1 (availability)
  │
  └── on sustained outage → Gemini Flash (cost)
```

Resiliência contra outage de provider único.

### Padrão 3: especialização por capability

```text
Task router
  ├── Code → Claude
  ├── Image → Gemini
  ├── Voice → GPT-4o
  ├── Math → o3
  └── Default → Claude Sonnet
```

Cada task para o modelo onde ele brilha.

### Padrão 4: self-host + cloud híbrido

```text
Baseline: Llama 3 self-host (custo zero após hardware)
  │
  └── Casos difíceis → Claude/GPT API
```

Custo previsível + qualidade em edge cases.

## Armadilhas comuns

### 1. "Claude é sempre melhor" / "GPT é sempre melhor"

Dogma. Nenhum modelo domina em tudo. **Fix:** benchmark próprio.

### 2. Benchmark público como verdade

MMLU, HumanEval, GSM8K são úteis mas não refletem seu caso. **Fix:** golden set seu.

### 3. Escolher por preço sem medir qualidade

Flash-Lite é barato, mas se ele quebra 20% das tasks que Sonnet resolve, custo real é maior. **Fix:** cost per correct output, não per token.

### 4. Ignorar custo de migração

"Vamos trocar para modelo X" sem considerar: re-tuning de prompts, re-eval, retreinamento de equipe, risco. **Fix:** incluir switching cost.

### 5. Lock-in por prompt engineering

Prompts muito específicos para um modelo dificultam migração. **Fix:** prompts portáveis + testes em múltiplos modelos.

### 6. Confundir modelo com ferramenta

"Cursor é melhor que Claude Code" — mas Cursor pode rodar Claude. Comparar apples-to-apples. **Fix:** separar dimensões.

### 7. Seguir hype semanal

"Novo modelo X bate tudo!" — probavelmente não no seu caso. **Fix:** revisar decisões a cada 3-6 meses, não a cada lançamento.

### 8. Ignorar regional pricing e latência

API em região errada = latência e compliance issues. **Fix:** considerar região desde início.

### 9. Budget único "IA"

Sem breakdown por feature, não consegue otimizar. **Fix:** metadata por chamada + dashboards.

### 10. Falha em validar após update

Provider atualiza modelo silenciosamente, prompt quebra. **Fix:** pin version + golden set em CI.

## Na prática (da minha experiência)

No MedEspecialista, fui passando por várias revisões de stack ao longo dos anos. O que decidiu cada vez:

**2023 — OpenAI primeira escolha.** Começamos com GPT-3.5 Turbo porque era o que tinha API estável e documentação boa. Funcionou para MVPs, mas qualidade em análise de prontuários era decepcionante.

**2024 — Migração para Claude como default.** Testes lado a lado mostraram qualidade superior do Claude 2 em análise médica longa. Migrei features críticas para Anthropic API mantendo GPT para fallback.

**2025 — Tiering aggressivo multi-provider.** Implementei padrão: Haiku para triagem, Sonnet para casos normais, Opus para edge cases. Custo total caiu ~60% sem perda de qualidade. Gemini Flash entrou para uma feature específica (triagem de mensagens de paciente) onde era objetivamente mais barato com qualidade equivalente.

**2026 — Multi-provider por capability.** Hoje:

- **Claude Sonnet 4.6** — feature principal (análise de prontuário, Q&A sobre docs médicas).
- **Claude Haiku** — triagem rápida em vários pontos.
- **Gemini 2.5 Pro** — análise multimodal de exames com imagem.
- **OpenAI GPT-4o** — voice/conversation em uma feature específica.
- **GPT-4.1-mini** — fallback quando Anthropic tem problemas.

Cada feature tem métricas próprias. Revisão a cada 6 meses, às vezes mais quando aparece modelo importante.

**Lições:**

- **Não há religião.** Use o melhor para cada caso, não o "favorito".
- **Tiering é obrigatório em escala.** Sem ele, custo explode.
- **Fallback multi-provider salva seu pescoço.** Outages acontecem.
- **Pin versions e tenha golden sets.** Updates silenciosos existem.
- **Meça custo por outcome**, não por token. Um modelo "caro" que acerta mais pode ser mais barato no total.

**Incidente memorável:** em 2025, um dia de outage da Anthropic (~3h em horário de pico). Features principais ficaram fora. Desde então: fallback automático para GPT-4.1 configurado em camada de roteamento, reduzindo RTO de 3h para segundos. Custo extra: mínimo, porque só é usado em fallback. **Lição:** resilience é design, não sorte.

## How to explain in English

### Short pitch

"There is no 'best LLM'. There's the best LLM for a given task, budget, latency target, stack, and compliance constraints. I maintain a multi-provider stack — Claude as the primary, Gemini for multimodal, GPT for voice and fallback — and I tier aggressively so that Haiku or Flash handles the simple cases and the big models only get the hard ones. That's what keeps cost and quality in balance at scale."

### Deeper version

"The core mental model I apply: separate model from tool, tier by task, always benchmark on my data. Separate model from tool because 'Claude Code' and 'Claude' are different things — one is a coding agent, the other is a model. Tier by task because using Opus or GPT-o3 for everything is wasteful; Haiku and Flash handle simple classification at a fraction of the cost. And benchmark on my data because public benchmarks don't reflect how my features actually behave.

The current meta in 2026: Claude leads in reasoning and tool use, Gemini in native multimodal and cost at the low end, GPT in ecosystem breadth and voice. For a greenfield project, I'd typically start with Claude Sonnet as the main model, add Haiku for triage, and bring in Gemini specifically for multimodal or extreme-cost use cases. I'd use Claude Code as my coding agent paired with GitHub Copilot for IDE completions.

For enterprise I consider platform choice alongside model choice — Azure OpenAI, Vertex AI, or Bedrock — because data residency, IAM, and audit logs matter more than the model at that level."

### Talking points

- "No best model — best model for the specific (task, budget, latency, compliance) combination."
- "Tiering is not an optimization, it's the default."
- "Pin model versions in production. Golden sets on every change."
- "Fallback multi-provider saves you in outages. One incident teaches the lesson."
- "Separate model choice from tool choice."

### Key vocabulary

- estratégia multi-provedor → multi-provider strategy
- tier de modelo → model tier
- fallback → fallback
- conjunto dourado → golden set
- custo por resultado correto → cost per correct output
- preço por região → regional pricing
- custo total de propriedade → total cost of ownership (TCO)
- fidelidade de saída → output faithfulness
- ecossistema de ferramentas → tooling ecosystem
- lock-in → vendor lock-in

## Recursos

### Benchmarks e leaderboards

- [LMSYS Chatbot Arena](https://chat.lmsys.org/) — ranking por voto humano
- [Artificial Analysis](https://artificialanalysis.ai/) — benchmarks independentes, cost/quality
- [MTEB](https://huggingface.co/spaces/mteb/leaderboard) — embeddings
- [HumanEval, MBPP, LiveCodeBench](https://paperswithcode.com/task/code-generation) — coding benchmarks
- [MMLU, GPQA, AGIEval](https://paperswithcode.com/sota) — reasoning benchmarks

### Análises independentes

- [Simon Willison's weblog](https://simonwillison.net/)
- [Pragmatic Engineer — AI](https://newsletter.pragmaticengineer.com/)
- [Ethan Mollick — One Useful Thing](https://www.oneusefulthing.org/)
- [Latent Space podcast](https://www.latent.space/)

### Pricing trackers

- [Artificial Analysis pricing](https://artificialanalysis.ai/models)
- [OpenRouter](https://openrouter.ai/models) — pricing + ao vivo

### Documentação oficial (pricing)

- [Anthropic Pricing](https://www.anthropic.com/pricing)
- [OpenAI Pricing](https://openai.com/pricing)
- [Gemini Pricing](https://ai.google.dev/pricing)
- [AWS Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/)
- [Azure OpenAI Pricing](https://azure.microsoft.com/en-us/pricing/details/cognitive-services/openai-service/)
- [Vertex AI Pricing](https://cloud.google.com/vertex-ai/pricing)

## Veja também

- [[LLMs]] — fundamentos técnicos
- [[Claude]] — ecossistema Anthropic
- [[GitHub Copilot]] — assistente integrado ao IDE
- [[Codex]] — agent cloud da OpenAI
- [[Gemini]] — família Google
- [[Skills e Prompting]] — que funciona cross-model
- [[Agents]] — padrões de agent independente de modelo
- [[RAG e Vector Databases]]
- [[MCP]]
- [[Inteligência Artificial]]
- [[Trilha IA]]
