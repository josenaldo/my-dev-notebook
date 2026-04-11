---
title: "Agents"
created: 2026-04-01
updated: 2026-04-11
type: concept
status: evergreen
tags:
  - ia
  - agents
  - entrevista
  - fundamentos
publish: true
---

# Agents

> Agents são o nível mais alto de abstração em aplicações com LLM em 2026. A diferença entre um chatbot e um agent é a mesma entre uma função pura e um programa inteiro: agents **raciocinam, decidem, usam ferramentas, observam resultados, e iteram** até terminar a tarefa. Esta nota é a trilha completa — do conceito fundamental até construir agents de produção com observabilidade, guardrails e evaluation. É também a área onde mais se vê over-engineering: muita gente usa agent onde um prompt direto bastaria. Um senior sabe onde a complexidade vale a pena.

## O que é

Um **AI Agent** é um sistema que combina um LLM (o "cérebro") com um conjunto de ferramentas (APIs, código, busca, leitura de arquivos) e um loop de execução. Dado um objetivo, o agent **decide sozinho** o que fazer em cada passo: qual ferramenta chamar, com quais argumentos, quando pedir mais informação, quando terminar.

Diferente de:

- **Chat simples:** uma pergunta, uma resposta. Sem ferramentas, sem iteração.
- **Prompt com RAG:** pipeline fixo (busca → contexto → resposta). Sem decisão dinâmica.
- **Workflow hardcoded:** sequência fixa de chamadas LLM. Sem autonomia.

Um agent tem **autonomia de decisão** no loop. Isso é o que o define.

### Anatomia mínima

```text
┌────────────────────────────────┐
│          AGENT LOOP            │
│                                │
│  1. Receber objetivo           │
│       │                        │
│       ▼                        │
│  2. LLM decide próxima ação    │◄──┐
│       │                        │   │
│       ▼                        │   │
│  3. Executar tool call         │   │
│       │                        │   │
│       ▼                        │   │
│  4. Observar resultado         │   │
│       │                        │   │
│       ▼                        │   │
│  5. Decidir: continuar ou      │   │
│     terminar?                  │───┘
│       │                        │
│       ▼                        │
│  6. Retornar resultado final   │
└────────────────────────────────┘
```

## O que diferencia um senior em agents

1. **Sabe quando NÃO usar agent.** A maior parte das tarefas simples é resolvida com prompt + ferramenta direta.
2. **Desenha tools como APIs de verdade:** descrições claras, tipos, erros úteis, sem sobreposição.
3. **Sempre define `max_steps` e guardrails.** Agent sem limite é bomba-relógio de custo.
4. **Trata ações destrutivas com human-in-the-loop.** Delete, merge, deploy, email externo = confirmação obrigatória.
5. **Instrumenta tudo.** Cada tool call, input, output, latência, custo. Agent sem observabilidade = debugging às cegas.
6. **Entende que agents falham de formas novas** — loop infinito, decisão errada, tool usage incorreto — e desenha para isso.
7. **Decompõe em sub-agents** quando a tarefa é complexa, em vez de um agent gigante.
8. **Mede resultado, não processo.** Agent que termina em 10 steps com resultado certo > agent que termina em 3 steps com resultado errado.
9. **Pratica prompt injection defense.** Conteúdo externo (web, docs) é adversarial por padrão.
10. **Tem evaluation de agent**, não só evaluation de LLM. Métricas: task completion rate, cost per task, error rate, human intervention rate.

## Trilha de aprendizado — do zero ao domínio de agents

### Nível 1 — Entender o conceito (1 semana)

- Use Claude Code ou Cursor em tarefas reais. Note quando ele decide bem, quando decide mal.
- Leia [Building Effective Agents — Anthropic](https://www.anthropic.com/research/building-effective-agents).
- Distinga agent de chatbot, de workflow, de pipeline RAG.

**Check:** você consegue explicar o loop ReAct e por que agents são diferentes de chamadas LLM diretas.

### Nível 2 — Construir um agent simples (1-2 semanas)

- Implemente manualmente (sem framework) um agent com tool use: pergunta → pensa → chama ferramenta → observa → responde.
- 2-3 ferramentas: calculadora, web search, file read.
- Sem framework, raw SDK (Anthropic ou OpenAI).
- `max_steps=10`, logging de cada passo.

**Check:** você construiu um agent do zero e entende exatamente o que acontece em cada iteração.

### Nível 3 — Ferramentas e guardrails (2-3 semanas)

- Adicione ferramentas complexas: file write, shell execution, API chamadas.
- Implemente human-in-the-loop para ações destrutivas.
- Sandboxing: rodar código em ambiente isolado.
- Error recovery: retry, fallback, gracefully skipping.

**Check:** você consegue dar a um agent ferramentas poderosas sem ele causar estragos.

### Nível 4 — Frameworks e produção (3-6 semanas)

- Use um framework de verdade: Claude Agent SDK, LangChain/LangGraph, Vercel AI SDK, CrewAI.
- Multi-agent: orchestrator + sub-agents especializados.
- Memory: curto e longo prazo, structured note-taking.
- Planning: decomposição de tarefas complexas.
- Observabilidade: Langfuse/LangSmith integrados.

**Check:** você tem um agent em produção que você consegue operar com confiança.

### Nível 5 — Evaluation e ops (ongoing)

- Golden set de tarefas + grading (match semântico, LLM-as-judge, human eval).
- A/B test de estratégias de agent.
- Cost budgets e circuit breakers.
- Incident response playbook quando agent faz algo estúpido.
- Security review: prompt injection, exfiltration, privilege escalation.

**Check:** você assume ownership de um agent em produção com múltiplos usuários.

## Ciclo de um agent — o padrão ReAct em detalhe

**ReAct** (Reasoning + Acting), introduzido em 2022, virou o padrão mental de agents. Combina:

- **Reasoning traces** — "thoughts" onde o modelo raciocina sobre o que fazer.
- **Actions** — chamadas a ferramentas.
- **Observations** — resultado da ferramenta, alimentado de volta.

```text
Objective: "Find the 5 most recent papers about context engineering
and summarize them."

Thought: I need to search for recent papers on this topic.
Action: web_search(query="context engineering LLM agents 2025", limit=10)
Observation: [lista de 10 resultados]

Thought: I have candidates. I need to read the top 5 and summarize.
Action: read_url(url="https://arxiv.org/abs/2506.12345")
Observation: [conteúdo do paper 1]

Thought: Good, this is relevant. Move to paper 2.
Action: read_url(url="...")
Observation: [conteúdo do paper 2]

... (repeat for papers 3-5)

Thought: I have enough. Let me synthesize.
Final answer: [sumário estruturado dos 5 papers]
```

Em 2026, quase ninguém mais formata ReAct textualmente — LLMs modernos têm **native tool use**, e o loop é gerenciado pelo SDK. Mas o mental model é o mesmo.

### Native tool use com Claude

```python
from anthropic import Anthropic

client = Anthropic()

tools = [
    {
        "name": "web_search",
        "description": "Search the web for recent articles and pages",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string"},
                "limit": {"type": "integer", "default": 10}
            },
            "required": ["query"]
        }
    },
    {
        "name": "read_url",
        "description": "Fetch and return the textual content of a URL",
        "input_schema": {
            "type": "object",
            "properties": {"url": {"type": "string"}},
            "required": ["url"]
        }
    }
]

def run_agent(objective: str, max_steps: int = 15):
    messages = [{"role": "user", "content": objective}]
    for step in range(max_steps):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason == "end_turn":
            # Agent decidiu terminar
            final_text = next(
                (b.text for b in response.content if b.type == "text"), ""
            )
            return final_text

        # Processar tool calls
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                result = execute_tool(block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result
                })
        messages.append({"role": "user", "content": tool_results})

    raise RuntimeError("Max steps exceeded")

def execute_tool(name: str, inputs: dict) -> str:
    if name == "web_search":
        return json.dumps(web_search(**inputs))
    if name == "read_url":
        return fetch_url(**inputs)
    raise ValueError(f"Unknown tool: {name}")
```

Esse é o "hello world" de um agent. De verdade. Todos os frameworks são variações disso com açúcar sintático, observabilidade, memory, etc.

## Componentes essenciais

### 1. LLM — o cérebro

Qualquer modelo capaz de tool use pode servir. Na prática:

- **Claude (Sonnet/Opus 4+):** top em tool use, raciocínio, seguir instruções complexas.
- **GPT-4.1 / o1:** strong, ecossistema maior.
- **Gemini 2.5 Pro:** multimodal nativo, contexto enorme.
- **Haiku/Flash/GPT-4o-mini:** para sub-agents simples (triagem, extração).

**Tiering:** use modelos pequenos para passos simples, grandes para passos decisivos.

### 2. Tools — as mãos

Tools são o que transforma um LLM em agent. Design é crítico.

**Princípios de tool design:**

- **Nome claro e único.** `search_docs` > `search`. Sem ambiguidade.
- **Descrição como API docstring.** O que faz, quando usar, que inputs espera, que retorna.
- **Inputs tipados com schema.** Cada parâmetro com tipo, descrição, default quando possível.
- **Outputs compactos e estruturados.** JSON quando possível. Textos curtos e focados.
- **Erros informativos.** "Invalid date format. Expected YYYY-MM-DD, got '12/01/2026'" > "Error".
- **Sem sobreposição.** Duas tools que fazem coisa similar confundem o modelo.
- **Idempotência quando possível.** Chamar 2x não causa estrago.

**Categorias comuns de tools:**

| Categoria | Exemplos |
| --- | --- |
| **Read-only** | web_search, read_file, query_db, get_weather |
| **Write local** | write_file, edit_file, run_shell_command |
| **Write external** | send_email, create_issue, post_to_api |
| **Interactive** | ask_user, request_confirmation |
| **Meta** | list_tools, get_schema, log_observation |

**Tools destrutivas** (delete, drop, overwrite, push, send external) precisam sempre de human-in-the-loop ou sandboxing.

### 3. Memory — o que persiste

Agents precisam lembrar coisas através de passos. Tipos de memory:

**Short-term (working memory):** contexto da conversa atual, carregado no prompt. Limitado pelo context window. Desaparece ao terminar.

**Long-term (persistent memory):** info que persiste entre sessões. Normalmente em arquivo ou DB externo.

- **File-based:** agent grava notas em arquivos markdown (ex: pasta `memory/` do Claude Code). Simples, inspecionável.
- **Vector store:** embeddings de memórias passadas, recuperadas por similaridade. Escalável mas menos controlável.
- **Structured DB:** tabelas com entidades, relações, factos. Máxima estrutura.

**Working memory compaction:** quando conversa fica longa, agent resume histórico e substitui por resumo. Ver [[Skills e Prompting]] para context engineering.

### 4. Planning — quebrar tarefas

Para tarefas complexas, agents podem beneficiar de plano explícito antes de executar.

**Strategies:**

- **Plan-then-execute:** agent gera plano em markdown, revisa, depois executa. Mais controlável.
- **Dynamic planning:** agent decide próximo passo em cada iteração. Mais flexível, menos previsível.
- **Hierarchical:** plano alto-nível, sub-planos por etapa.

**Claude Code prática:** eu sempre peço para Claude escrever plano em markdown antes de tocar em código para features não-triviais. Revisamos juntos, depois executa. Isso elimina ~80% dos "o Claude foi fazer outra coisa".

### 5. Orchestrator + sub-agents

Para tarefas grandes, um **agent orchestrator** delega para **sub-agents especializados**.

```text
Orchestrator (task planner)
├── Explorer (entende o problema)
├── Planner (propõe abordagens)
├── Implementer (escreve código)
├── Reviewer (revisa código)
└── Tester (roda testes)
```

**Vantagens:**

- Cada sub-agent tem contexto pequeno e focado.
- Falhas localizadas não contaminam o todo.
- Paralelização: múltiplos sub-agents independentes rodam concorrentes.
- Especialização por modelo: Explorer pode ser Haiku, Implementer Sonnet, Reviewer Opus.

**Desvantagens:**

- Overhead de coordenação.
- Handoff é onde informação se perde.
- Debugging mais complexo.

Use sub-agents quando tarefa não cabe em um contexto só, ou quando especialização claramente ajuda.

## Patterns comuns de agents

### Pattern 1: Tool-using assistant

Agent único com ferramentas read-only, usado para Q&A e análise.

**Exemplos:** pesquisa, análise de logs, Q&A sobre documentação interna.

**Guardrails:** nenhum crítico porque tools são safe.

### Pattern 2: Coding agent (local)

Agent que edita código, roda testes, faz commits. Opera no filesystem local do usuário.

**Exemplos:** Claude Code, Cursor, Cline, Aider.

**Tools:** read_file, write_file, run_shell, git_commit, etc.

**Guardrails:**

- `max_steps` alto mas finito (30-50).
- Confirmação antes de `git push`, `rm -rf`, operations fora do workspace.
- Sandboxing quando possível.
- Human review antes de merge.

### Pattern 3: Cloud agent (async)

Agent rodando em sandbox cloud, recebe task, retorna PR.

**Exemplos:** OpenAI Codex, Devin.

**Vantagens:** não afeta ambiente local; pode rodar em paralelo.

**Desvantagens:** menos interativo; harder to steer quando vai pelo caminho errado.

### Pattern 4: RAG agent

Pipeline RAG com agent decidindo quando buscar, o quê buscar, quando expandir busca.

**Exemplos:** agent de pesquisa que usa web search + read, agent de Q&A sobre base de conhecimento.

**Tools:** search (vector ou web), rerank, read_doc.

**Diferença de RAG "pipeline fixo":** o agent decide iterativamente se o contexto obtido é suficiente.

### Pattern 5: Multi-agent orchestration

Múltiplos agents especializados coordenados por orchestrator. Usado em workflows complexos.

**Frameworks:** CrewAI, AutoGen, LangGraph, Claude Agent SDK.

**Use case:** geração de conteúdo (pesquisa → draft → revisão → publicação), análise de dados (fetch → clean → analyze → visualize → report).

### Pattern 6: Workflow híbrido (workflow + agent)

Workflow determinístico com steps que são prompts LLM ou pequenos agents. Muita gente chama de "agent" mas é mais workflow.

**Quando usar:** quando o processo é previsível e estável. Mais barato, mais debugável.

**Regra da Anthropic:** "use workflows when you can, agents when you must."

## Frameworks em 2026

### Claude Agent SDK

Framework oficial da Anthropic para construir agents com Claude.

- **Prós:** integração nativa com Claude, tool use otimizado, observabilidade built-in, support a MCP.
- **Contras:** lock-in Anthropic (até pode usar outros modelos, mas é otimizado para Claude).
- **Quando usar:** se você está comprometido com Claude e quer o melhor do Claude.

[Claude Agent SDK Docs](https://docs.anthropic.com/en/docs/agents)

### LangChain / LangGraph

O framework mais popular para agents em Python/JS.

- **LangChain:** abstrações de "chain" e "agent", muitas integrações.
- **LangGraph:** layer para grafos de execução, stateful workflows, cycles, branches.
- **Prós:** ecossistema enorme, integra com tudo, LangSmith para observabilidade.
- **Contras:** abstrações pesadas, mudanças frequentes, debugging difícil.
- **Quando usar:** projetos que precisam de múltiplas integrações e estão ok com overhead.

### Vercel AI SDK

Framework para aplicações Next.js/React com LLM.

- **Prós:** excelente DX em frontend, streaming nativo, hooks React.
- **Contras:** focado em aplicações web, não para agents servidor-puros.
- **Quando usar:** SPA/webapp com IA.

### CrewAI

Framework especializado em multi-agent orchestration.

- **Prós:** paradigma claro de "crew" (papéis + tarefas), boa para ideação.
- **Contras:** menos maduro, documentação variável.
- **Quando usar:** protótipos multi-agent.

### AutoGen (Microsoft)

Framework de multi-agent conversational.

- **Prós:** paradigma claro, support a human-in-the-loop.
- **Contras:** mais acadêmico, menos otimizado para prod.

### Frameworks "sem framework"

Em 2026, muita gente está voltando para **SDK raw + código próprio**. Motivo: frameworks são abstrações que engessam. Um agent de 500 linhas em TypeScript raw é:

- Mais fácil de debugar
- Mais fácil de otimizar custo
- Mais fácil de adaptar

Se você sabe o que está fazendo e tem use case estável, considere não usar framework. Esta é a abordagem de Simon Willison e vários devs senior.

## Guardrails — o que impede agents de causar estragos

### 1. Max steps

**Obrigatório.** Sempre. `max_steps = 20` (ou o que fizer sentido). Agent não termina = exceção.

### 2. Cost budget

`max_cost_usd = 1.00` por task. Quando excede, interrompe. Critical em prod.

### 3. Tool allowlisting

Só ferramentas necessárias para a task. Agent de research não precisa de `write_file`.

### 4. Human-in-the-loop para ações destrutivas

- `git push` → confirma
- `rm -rf` → confirma
- `DROP TABLE` → confirma
- `send_email` → confirma (a não ser que seja use case explícito)

### 5. Sandboxing

Código que o agent executa → sandbox (Docker, gVisor, Firecracker, nsjail). **Nunca** execute LLM-generated code no seu host sem camada de isolação.

### 6. Output validation

Validar tool inputs e outputs. Exemplo: agent retornou caminho com `../` → reject.

### 7. Rate limiting

Por usuário, por task, por tool. Evita abuse e runaway loops.

### 8. Kill switch

Capability para interromper um agent em runtime. Importante para sistemas 24/7.

### 9. Audit log

Log estruturado de cada tool call, input, output, user ID, timestamp. Obrigatório em qualquer sistema com ações externas.

### 10. Confirmation para out-of-scope

Se o agent decide fazer algo fora do escopo declarado, pedir confirmação.

## Evaluation de agents

Agents são harder de avaliar que LLMs puros porque o processo é não-determinístico e tem múltiplos steps.

### Métricas fundamentais

- **Task completion rate:** % de tasks que terminaram com resultado correto.
- **Steps per task:** eficiência.
- **Cost per task:** $/task.
- **Latency (p50, p99):** quanto demora.
- **Human intervention rate:** quantas vezes humano precisou intervir.
- **Error types:** catalogar falhas (loop, wrong tool, wrong argument, etc).

### Golden set de tasks

30-100 tasks representativas com resultado esperado. Rode a cada mudança significativa.

### LLM-as-judge

Para resultados abertos, use modelo forte para avaliar output do agent. Prompt do judge cobre: correção, completude, eficiência.

### Trace review

Revisão humana de traces de agent em produção. Invista 1-2h/semana lendo traces — você vai encontrar bugs que nenhum eval automatizado pega.

### Regression tests

Quando um bug é encontrado, adicione ao golden set. Nunca perca a mesma regressão duas vezes.

## Armadilhas comuns

### 1. Usar agent onde prompt bastava

Over-engineering clássico. "Vamos fazer um agent!" para algo que é `LLM(input) → output`. **Regra:** se a tarefa não precisa de decisão iterativa ou uso de ferramentas, não é agent.

### 2. Sem max_steps

Agent entra em loop, consome $50 de API em 10 minutos. **Fix:** `max_steps` sempre.

### 3. Tool description ruim

"search — searches for things." Agent não sabe quando usar. **Fix:** descrição clara, exemplos de quando usar e não usar.

### 4. Tool outputs gigantes

Tool retorna 50K tokens de JSON. Contexto estoura, agent fica confuso. **Fix:** compactar outputs, paginar, retornar só o relevante.

### 5. Tools sobrepostas

`search` e `query` e `find`. Agent não sabe qual usar. **Fix:** consolidar.

### 6. Sem sandboxing para code execution

Agent executa código no seu host, `rm -rf /` não é piada. **Fix:** Docker/gVisor sandbox, allowlist de comandos.

### 7. Prompt injection ignorado

Agent lê conteúdo de URL → URL contém instruções maliciosas → agent obedece. **Fix:** delimitação XML, tool allowlist conservador, human-in-the-loop.

### 8. Sem observabilidade

Agent falha em produção, você não sabe por quê. **Fix:** tracing de cada tool call, input, output.

### 9. Multi-agent prematuro

"Vamos fazer 5 agents especializados." Para uma task que um agent resolve. Overhead desnecessário. **Fix:** começar com 1 agent, só decompor quando justificado.

### 10. Avaliar só com eyeballing

"Testei 5 tasks, parece ok." **Fix:** golden set, métricas, revisão de traces regular.

## Exemplo completo — agent de research

```python
import json
from anthropic import Anthropic
from dataclasses import dataclass

client = Anthropic()

tools = [
    {
        "name": "web_search",
        "description": (
            "Search the web for recent pages. Use when you need current "
            "information or sources for a claim. Returns title, url, snippet."
        ),
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"},
                "limit": {"type": "integer", "default": 10, "maximum": 20}
            },
            "required": ["query"]
        }
    },
    {
        "name": "read_url",
        "description": (
            "Fetch the textual content of a specific URL. Use after "
            "web_search to read the most promising results. Returns plain text."
        ),
        "input_schema": {
            "type": "object",
            "properties": {"url": {"type": "string"}},
            "required": ["url"]
        }
    },
    {
        "name": "record_finding",
        "description": (
            "Record an important finding with source. Use each time you "
            "discover relevant information to keep structured notes."
        ),
        "input_schema": {
            "type": "object",
            "properties": {
                "claim": {"type": "string"},
                "source_url": {"type": "string"},
                "confidence": {"type": "string", "enum": ["high", "medium", "low"]}
            },
            "required": ["claim", "source_url", "confidence"]
        }
    }
]

SYSTEM = """You are a research agent. Your task is to thoroughly research
a topic using web search and reading, then synthesize findings.

Process:
1. Start with web_search to find candidate sources
2. read_url for the most promising ones
3. Use record_finding to track each important fact with source
4. When you have enough findings (typically 5-10), synthesize final answer

Rules:
- Always cite sources
- Prefer primary sources over summaries
- If sources contradict, note the disagreement
- Stop when you have enough — do not over-search
"""

@dataclass
class AgentResult:
    answer: str
    steps: int
    findings: list
    cost_usd: float

def run_research_agent(query: str, max_steps: int = 15) -> AgentResult:
    findings = []
    messages = [{"role": "user", "content": f"Research: {query}"}]
    total_input_tokens = 0
    total_output_tokens = 0

    for step in range(max_steps):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            system=SYSTEM,
            tools=tools,
            messages=messages
        )
        total_input_tokens += response.usage.input_tokens
        total_output_tokens += response.usage.output_tokens
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason == "end_turn":
            final = next(
                (b.text for b in response.content if b.type == "text"), ""
            )
            cost = estimate_cost(total_input_tokens, total_output_tokens)
            return AgentResult(final, step + 1, findings, cost)

        tool_results = []
        for block in response.content:
            if block.type != "tool_use":
                continue
            try:
                if block.name == "web_search":
                    result = json.dumps(web_search_impl(**block.input))
                elif block.name == "read_url":
                    result = fetch_url(**block.input)[:8000]  # truncate
                elif block.name == "record_finding":
                    findings.append(block.input)
                    result = "recorded"
                else:
                    result = f"Unknown tool: {block.name}"
            except Exception as e:
                result = f"Tool error: {e}"
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": block.id,
                "content": result
            })
        messages.append({"role": "user", "content": tool_results})

    raise RuntimeError(f"Max steps ({max_steps}) exceeded")

def estimate_cost(input_tokens: int, output_tokens: int) -> float:
    # Claude Sonnet 4.6 pricing: $3/M in, $15/M out
    return (input_tokens * 3 + output_tokens * 15) / 1_000_000
```

Esse é um agent real, funcional, com:

- Max steps
- Tool descriptions claras
- Compact outputs
- Error handling no tool execution
- Structured findings via tool dedicada
- Cost tracking
- System prompt explicitando processo

Qualquer coisa além disso é otimização.

## Na prática (da minha experiência)

No MedEspecialista, minhas experiências com agents foram uma progressão:

**Tentativa 1 (2023) — agent genérico com LangChain.** Instalei LangChain, segui tutorial, fiz um "agent que pode fazer tudo". Resultado: lento, caro, imprevisível, debugging via print-statements. Abandonado em 1 mês.

**Tentativa 2 (2024) — agents especializados, frameworks limitados.** Parei de tentar fazer "o agent que resolve tudo" e comecei a fazer agents específicos para workflows específicos: um para triagem de tickets, um para sumarização de consultas, um para geração de relatórios. Cada um com 3-5 tools focadas. Muito mais estável.

**Tentativa 3 (2025-2026) — Claude Code como agent core + sub-agents especializados.** Hoje meu workflow principal de coding é Claude Code com skills customizadas + sub-agents (explorer, planner, reviewer) para tarefas grandes. Outros workflows (não-coding) uso SDK raw + código próprio, sem framework.

**Lições:**

- **Começar simples, adicionar complexidade só quando dói.** 90% do que se chama "agent" funciona melhor como workflow determinístico com steps LLM.
- **Tool descriptions são 60% do trabalho.** Investir tempo escrevendo descrições boas é o maior ROI.
- **Human-in-the-loop é feature, não limitação.** Usuários preferem agent que pergunta antes de deletar.
- **Observabilidade desde o dia 1.** Não tem como debugar agent sem traces.
- **Compactação de output de tools é subestimada.** Agent trabalha melhor com tool que retorna 500 tokens do que com tool que retorna 5000.
- **Single agent bem desenhado > multi-agent confuso.**

**Incidente memorável:** agent de refactor com tool `replace_in_files` sem confirmação causou um `git reset --hard` de 2h de trabalho quando entrou em loop renomeando símbolos. Desde então: (a) `max_steps` sempre; (b) confirmação humana antes de mudanças em > 5 arquivos; (c) dry-run mode para refactors; (d) nunca agent escreve no filesystem sem git clean state ou worktree.

**Outro incidente:** agent de suporte começou a receber mensagens com prompt injection: usuários descobriram que podiam fazer o agent "esquecer" as regras e executar ações privilegiadas. Fix: separação clara via tags `<user_message>...</user_message>`, tool allowlist conservador (nenhuma tool destrutiva), e um second-pass LLM que classifica respostas antes de enviar ao usuário.

## How to explain in English

### Short pitch

"Agents are LLM systems with tools and a decision loop. They reason about what to do, call tools, observe results, and iterate until done. My approach is: use workflows whenever possible, agents when you really need dynamic decision-making. Every agent I build has max_steps, observability, and guardrails for destructive actions — those aren't optional in production."

### Longer version

"I think about agents as stochastic, partially-autonomous programs. They're harder to build, debug, and operate than regular code. The rule I live by is Anthropic's: 'use workflows when you can, agents when you must'.

When I do build agents, the fundamentals matter more than the framework. Clear tool descriptions with good input/output schemas. Compact tool outputs that don't pollute context. Max_steps always. Cost budgets always. Human-in-the-loop for destructive actions. Sandboxing for code execution. Observability on every tool call.

My daily driver is Claude Code as a coding agent — I use it for pair programming, refactoring, and debugging. For non-coding workflows I typically write agents with the raw Anthropic SDK plus maybe 500 lines of my own scaffolding. I've been burned by heavy frameworks enough times that I prefer code I fully understand.

For production agents, evaluation is where the bar goes up. I maintain a golden set of representative tasks, measure completion rate, cost per task, latency, and human intervention rate. I review production traces weekly — that's where I find bugs evals miss."

### Phrases to use

- "Workflows when you can, agents when you must."
- "A tool without a clear description is worse than no tool at all."
- "Agents fail in new and creative ways. Design for that."
- "Prompt injection is the SQL injection of the LLM era. Assume adversarial input."
- "Observability is not optional. An agent without traces is a time bomb."
- "I don't use a framework unless the pain of not having one exceeds the pain of having one."

### Key vocabulary

- agente → agent
- loop de agente → agent loop
- ferramenta → tool
- uso de ferramenta → tool use / function calling
- raciocínio e ação → reasoning and acting (ReAct)
- planejamento → planning
- orquestração → orchestration
- sub-agente → sub-agent
- memória → memory
- memória persistente → persistent memory
- caixa de areia → sandbox
- humano no loop → human-in-the-loop
- passo máximo → max steps
- orçamento de custo → cost budget
- injeção de prompt → prompt injection
- chave de emergência → kill switch
- rastreamento → tracing

## Recursos

### Leitura fundamental

- [Building Effective Agents — Anthropic](https://www.anthropic.com/research/building-effective-agents) — o "must read"
- [Effective Context Engineering for AI Agents — Anthropic](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [12-Factor Agents](https://github.com/humanlayer/12-factor-agents)
- [Agentic Engineering — Addy Osmani](https://addyosmani.com/blog/agentic-engineering/)
- [Factory Model — Addy Osmani](https://addyosmani.com/blog/factory-model/)
- [The Third Golden Age of Software — Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/the-third-golden-age-of-software)
- [ReAct paper (arxiv)](https://arxiv.org/abs/2210.03629)
- [Simon Willison on agents](https://simonwillison.net/tags/llms/)

### Docs de frameworks

- [Claude Agent SDK](https://docs.anthropic.com/en/docs/agents)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [LangChain](https://python.langchain.com/docs/get_started/introduction)
- [LangGraph](https://langchain-ai.github.io/langgraph/)
- [Vercel AI SDK](https://sdk.vercel.ai/docs)
- [CrewAI](https://docs.crewai.com/)
- [AutoGen (Microsoft)](https://microsoft.github.io/autogen/)
- [Claude Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)

### Ferramentas do ecossistema

- [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) — curadoria massiva
- [Context Lens](https://github.com/TiagoSchr/context-lens) — análise de contexto
- [Junie — JetBrains AI](https://www.jetbrains.com/guide/ai/article/junie/)
- [Langfuse](https://langfuse.com/) — observabilidade de agents
- [LangSmith](https://smith.langchain.com/)
- [Full Cycle com Agents (playlist)](https://www.youtube.com/playlist?list=PLucm8g_ezqNoAkYKXN_zWupyH6hQCAwxY)

### Segurança

- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [Simon Willison — Prompt injection](https://simonwillison.net/series/prompt-injection/)

## Veja também

- [[Inteligência Artificial]]
- [[LLMs]] — os fundamentos que agents usam
- [[Skills e Prompting]] — prompt engineering e skills
- [[RAG e Vector Databases]] — RAG como componente de agent
- [[MCP]] — Model Context Protocol, o "USB-C para agents"
- [[Claude]]
- [[GitHub Copilot]]
- [[Codex]]
- [[Gemini]]
- [[Comparativo de LLMs]]
- [[Trilha IA]]
