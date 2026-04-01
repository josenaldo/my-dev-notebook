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

Ir além de prompt engineering — estruturar todo o contexto que o LLM recebe:

- **Documentação do projeto:** CLAUDE.md, README, ADRs
- **Código relevante:** arquivos que o model precisa entender
- **Restrições:** limites, convenções, padrões do projeto
- **Exemplos:** output esperado, formato, estilo

### Skills (Claude Code)

Skills são instruções reutilizáveis que definem comportamentos específicos:

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
