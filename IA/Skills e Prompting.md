---
title: "Skills e Prompting"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - ia
  - prompting
  - entrevista
publish: false
---

# Skills e Prompting

Prompt engineering e skills — como comunicar efetivamente com LLMs para obter resultados consistentes.

## O que é

Prompt engineering é a prática de estruturar instruções para LLMs de forma a obter respostas previsíveis e de alta qualidade. Skills são conjuntos reutilizáveis de instruções e comportamentos que estendem as capacidades de um agent.

## Como funciona

### Técnicas de prompting

**Zero-shot:** pergunta direta sem exemplos.

**Few-shot:** incluir exemplos do formato esperado.

```text
Classifique o sentimento: "O produto é excelente" → positivo
Classifique o sentimento: "Péssima experiência" → negativo
Classifique o sentimento: "O atendimento foi ok" → ?
```

**Chain-of-Thought (CoT):** pedir para o modelo raciocinar passo a passo.

```text
Pense passo a passo antes de responder:
1. Identifique os requisitos
2. Liste as opções
3. Analise trade-offs
4. Recomende a melhor opção
```

**System prompts:** definir persona, regras e contexto.

```text
system: Você é um arquiteto de software senior. Sempre considere 
escalabilidade, manutenibilidade e custo. Responda em português 
com termos técnicos em inglês.
```

### Context Engineering

Ir além de prompt engineering — estruturar todo o contexto que o LLM recebe. Segundo a Anthropic, context engineering é "curar e manter o conjunto ótimo de tokens durante a inferência do LLM".

**Técnicas principais:**

- **Compaction:** resumir conversas longas preservando decisões críticas. Quando o contexto se aproxima do limite, comprimir mantendo o essencial.
- **Structured note-taking:** agentes mantêm notas persistentes fora da context window, recuperáveis depois. Permite tracking de progresso sem sobrecarregar memória ativa.
- **Tool design token-eficiente:** ferramentas bem descritas, sem funcionalidades sobrepostas que causem ambiguidade.
- **System prompts em altitude certa:** específicos para orientar, mas flexíveis para heurísticas.
- **Sub-agent architectures:** dividir tarefas entre agentes especializados com contextos limpos individuais.
- **Documentação do projeto:** CLAUDE.md, AGENTS.md, README, ADRs — fornecem contexto persistente.

**Princípio central:** "faça a coisa mais simples que funciona" — o menor conjunto possível de tokens de alto sinal.

> **Fontes:**
> - [Effective Context Engineering for AI Agents — Anthropic](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
> - [Complete Guide to AGENTS.md](https://www.aihero.dev/a-complete-guide-to-agents-md)
> - [Agentic Engineering — Addy Osmani](https://addyosmani.com/blog/agentic-engineering/)
> - [Spec-Driven Development](https://www.infoq.com/articles/spec-driven-development/)

### Agent Skills

Skills são pacotes reutilizáveis que combinam instruções, scripts e referências para orientar agentes de IA. Usam **descoberta progressiva** — o agente carrega informações sob demanda (menu → submenu → detalhe), mantendo o contexto reduzido.

**Estrutura de um skill:**

```markdown
---
name: code-review
description: Review code for bugs, security, and best practices
---

## Instruções
1. Analise o código para bugs e vulnerabilidades
2. Verifique aderência aos padrões do projeto
3. Sugira melhorias com justificativa
4. Classifique issues por severidade
```

**Ferramentas que suportam skills:**

| Ferramenta | Formato de skills | Distribuição |
| --- | --- | --- |
| Claude Code | SKILL.md, skills/ | Local + plugins |
| GitHub Copilot | .github/copilot-instructions.md | Repositório |
| Codex (OpenAI) | AGENTS.md | Repositório |
| Cursor | .cursor/rules/ | Local |

**Distribuição:** `npx skills` permite compartilhar skills via GitHub — um "npm da IA". Skills referenciam documentação em vez de replicar conteúdo, evitando obsolescência.

**12 Factor Agents:** princípios para construir agents robustos (similar aos 12 fatores de apps):
- Agentes devem ser compostos de tools bem definidas
- Estado deve ser persistido fora do agente
- Cada step deve ser observável e debugável

> **Fontes:**
> - [Agent Skills in 2026 — Neon](https://neon.com/blog/agent-skills-in-2026)
> - [Equipping Agents with Skills — Claude Blog](https://claude.com/blog/equipping-agents-for-the-real-world-with-agent-skills)
> - [Agent Skills spec](https://agentskills.io/home)
> - [12 Factor Agents](https://github.com/humanlayer/12-factor-agents)
> - [Codex Skills](https://developers.openai.com/codex/skills/)
> - [Cursor Skills](https://cursor.com/docs/context/skills)
> - [Claude Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)

### Repositórios de skills

- [Anthropic Skills](https://github.com/anthropics/skills/tree/main/skills)
- [Awesome Copilot Skills](https://github.com/github/awesome-copilot/tree/main/skills)
- [CopilotSkills](https://github.com/Oxilith/CopilotSkills)
- [Antigravity Kit](https://github.com/vudovn/antigravity-kit) — [docs](https://antigravity-kit.vercel.app/docs)

## Quando usar

- **System prompts:** sempre, em qualquer integração com LLM
- **Few-shot:** quando o formato de saída é específico
- **Chain-of-Thought:** raciocínio complexo, análise de trade-offs
- **Skills:** comportamentos recorrentes em agents
- **Context engineering:** projetos com múltiplas sessões de IA

## Armadilhas comuns

- **Prompt vago:** "me ajude" vs "gere um endpoint REST em Spring Boot que..."
- **Contexto excessivo:** mais contexto ≠ melhor. Foco no relevante.
- **Não iterar:** prompt engineering é iterativo. Refinar baseado em resultados.
- **Ignorar system prompt:** é o lugar mais impactante para definir comportamento

## How to explain in English

"Prompt engineering is about communicating effectively with AI models. The key techniques I use are: system prompts to define behavior and constraints, few-shot examples to demonstrate expected output format, and chain-of-thought prompting for complex reasoning tasks.

Beyond individual prompts, I practice context engineering — structuring the entire context an AI receives, including project documentation, relevant code, and constraints. This is what makes the difference between generic AI output and responses that are tailored to your specific project."

### Key vocabulary

- engenharia de prompt → prompt engineering
- engenharia de contexto → context engineering
- prompt do sistema → system prompt
- poucos exemplos → few-shot prompting
- cadeia de pensamento → chain-of-thought (CoT)
- habilidade → skill: comportamento reutilizável de um agent

## Veja também

- [[Inteligência Artificial]]
- [[LLMs]]
- [[Agents]]
- [[Prompts]]
