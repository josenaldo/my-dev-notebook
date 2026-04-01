---
title: "Agents"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - ia
  - agents
  - entrevista
publish: false
---

# Agents

Sistemas de IA que usam LLMs para raciocinar, planejar e executar ações autonomamente usando ferramentas.

## O que é

Um AI Agent é um sistema que combina um LLM com a capacidade de usar ferramentas (APIs, código, busca) para completar tarefas complexas de forma autônoma. Diferente de um chatbot simples, um agent pode: planejar passos, executar ações, observar resultados, e ajustar sua abordagem.

## Como funciona

### Ciclo de um Agent

```text
1. Recebe objetivo do usuário
2. Raciocina sobre o que precisa fazer (planning)
3. Escolhe uma ferramenta (tool selection)
4. Executa a ferramenta (action)
5. Observa o resultado (observation)
6. Decide: precisa de mais passos? (loop ou finalizar)
7. Retorna resultado ao usuário
```

### Componentes

- **LLM (cérebro):** raciocina e toma decisões
- **Tools (ferramentas):** ações que o agent pode executar (buscar na web, rodar código, chamar APIs)
- **Memory:** contexto da conversa e resultados anteriores
- **Planning:** estratégia para quebrar tarefas complexas em passos

### Padrões de Agents

**ReAct (Reason + Act):** o agent alterna entre raciocínio e ação.

**Tool Use:** o LLM decide qual ferramenta usar e com quais parâmetros. A ferramenta executa e retorna resultado.

**Multi-Agent:** múltiplos agents especializados colaboram. Ex: um agent pesquisa, outro escreve código, outro revisa.

### Exemplos reais

| Agent | O que faz |
| --- | --- |
| Claude Code | Edita código, roda testes, gerencia git |
| GitHub Copilot Workspace | Planeja e implementa features |
| Devin | Desenvolvedor AI autônomo |
| AutoGPT | Agent genérico com ferramentas |

### Como construir agents

**Anthropic Claude:**

```python
# Tool Use com Claude API
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    tools=[{
        "name": "search_patients",
        "description": "Search patients by name or specialty",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string"},
                "specialty": {"type": "string"}
            }
        }
    }],
    messages=[{"role": "user", "content": "Find cardiology patients"}]
)
```

**Frameworks:**

- **Claude Agent SDK:** framework oficial da Anthropic para agents
- **LangChain:** framework popular, muitas integrações
- **CrewAI:** multi-agent orchestration
- **Vercel AI SDK:** agents em aplicações Next.js/React

## Quando usar

- **Tarefas multi-step:** pesquisar, analisar, e agir baseado nos resultados
- **Automação de workflows:** code review, deploy, testes
- **Assistentes especializados:** agents que entendem um domínio específico
- **Não usar:** tarefas simples onde um prompt direto resolve

## Armadilhas comuns

- **Agent loop infinito:** sem limites de iteração, o agent pode ficar preso. Sempre definir max_steps.
- **Over-autonomy:** agents fazendo ações destrutivas sem confirmação humana
- **Custo:** cada passo consome tokens. Agents complexos podem ficar caros rapidamente.
- **Tool design ruim:** ferramentas mal descritas = agent não sabe quando/como usar

## How to explain in English

"AI Agents represent the next evolution beyond simple LLM interactions. Instead of a single prompt-response cycle, an agent can plan a multi-step approach, use tools to gather information or take actions, and iterate based on results.

In my work, I use Claude Code as a coding agent — it can read files, edit code, run tests, and make git commits autonomously. The key to building effective agents is well-designed tools with clear descriptions, so the LLM can reason about when and how to use them.

The architecture is straightforward: an LLM as the reasoning engine, a set of tools it can invoke, and a loop that continues until the task is complete or a human checkpoint is reached."

### Key vocabulary

- agente → agent: sistema autônomo com LLM + ferramentas
- ferramenta → tool: ação que o agent pode executar
- planejamento → planning: quebrar tarefa em passos
- raciocínio → reasoning: pensar sobre o que fazer
- orquestração → orchestration: coordenar múltiplos agents
- loop de execução → execution loop: ciclo razão-ação-observação

## Recursos

- [Claude Agent SDK](https://docs.anthropic.com/en/docs/agents) — framework da Anthropic
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — coding agent
- [[Skills e Prompting]]

## Veja também

- [[Inteligência Artificial]]
- [[LLMs]]
- [[Claude]]
- [[Comparativo de LLMs]]
