---
title: "Inteligência Artificial"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - ia
  - entrevista
publish: false
---

# Inteligência Artificial

Visão geral de IA para desenvolvedores — de machine learning a GenAI, o que importa saber para usar e integrar.

## O que é

Inteligência Artificial é o campo que desenvolve sistemas capazes de realizar tarefas que normalmente requerem inteligência humana. Para um desenvolvedor fullstack, o que importa não é treinar modelos do zero, mas entender os conceitos para **integrar, usar e construir com IA**.

## Como funciona

### Hierarquia de conceitos

```text
Inteligência Artificial (campo amplo)
  └── Machine Learning (aprender com dados)
       ├── Supervised Learning (classificação, regressão)
       ├── Unsupervised Learning (clustering, dimensionalidade)
       ├── Reinforcement Learning (recompensa/punição)
       └── Deep Learning (redes neurais profundas)
            └── Generative AI (gerar conteúdo novo)
                 ├── LLMs (texto) → GPT, Claude, Gemini
                 ├── Diffusion Models (imagens) → DALL-E, Midjourney
                 └── Multimodal (texto + imagem + áudio)
```

### Conceitos essenciais para devs

**Tokens:** unidade de processamento dos LLMs. ~4 caracteres em inglês, ~3 em português. Determina custo e context window.

**Context Window:** quantidade de tokens que o modelo pode processar de uma vez. Claude: até 1M tokens. GPT-4: até 128K.

**Temperature:** controla a criatividade/aleatoriedade. 0 = determinístico, 1 = mais criativo.

**Fine-tuning vs RAG:**

- **Fine-tuning:** retreinar o modelo com dados específicos. Caro, mas muda o comportamento.
- **RAG (Retrieval-Augmented Generation):** buscar informação relevante e incluir no prompt. Mais barato, sem retreinar.

**Embeddings:** representação numérica (vetor) de texto. Permite busca semântica — encontrar textos com significado similar, não apenas palavras iguais.

### IA no desenvolvimento de software

| Área | Ferramentas | Exemplo |
| --- | --- | --- |
| Code completion | Copilot, Claude Code, Codex | Autocompletar código |
| Code review | Claude, Copilot | Análise de PRs |
| Debugging | Claude, ChatGPT | Explicar erros |
| Testing | Copilot, Claude | Gerar testes |
| Documentation | Claude, GPT | Gerar docs |
| Architecture | Claude, GPT | Brainstorming de design |

## Quando usar

- **LLMs em aplicações:** chatbots, sumarização, classificação de texto, tradução
- **RAG:** quando precisa de respostas baseadas em dados privados/atualizados
- **Code assistants:** pair programming, code review, geração de testes
- **Embeddings + Vector DB:** busca semântica, recomendações

## Armadilhas comuns

- **Confiar cegamente em output de LLM:** sempre validar código e fatos gerados
- **Ignorar custos de API:** tokens somam rápido. Monitorar uso e otimizar prompts
- **Fine-tuning quando RAG basta:** RAG é mais barato e flexível para a maioria dos casos
- **Não entender limitações:** alucinações, conhecimento com cutoff, viés dos dados de treino

## Na prática (da minha experiência)

> No MedEspecialista, operacionalizei GitHub Copilot com skills e prompts customizados para acelerar desenvolvimento, testes automatizados e code reviews. O workflow documentation-driven com Markdown permite que a IA entenda o contexto do projeto e gere código mais relevante. Isso reduziu significativamente o tempo de implementação.

## How to explain in English

"As a developer, I see AI as a force multiplier. I use LLMs daily — Claude for architectural brainstorming and code review, Copilot for code completion, and I've integrated AI features into applications using APIs.

The key concepts I focus on are: understanding token economics for cost management, choosing between RAG and fine-tuning based on the use case, and knowing the limitations of current models — particularly hallucinations and context window constraints.

For building AI-powered features, I follow the pattern of starting with prompt engineering, then adding RAG if the model needs domain-specific knowledge, and only considering fine-tuning if the behavior change is fundamental."

### Key vocabulary

- aprendizado de máquina → machine learning (ML)
- aprendizado profundo → deep learning (DL)
- IA generativa → generative AI (GenAI)
- modelo de linguagem grande → large language model (LLM)
- janela de contexto → context window
- geração aumentada por recuperação → retrieval-augmented generation (RAG)
- ajuste fino → fine-tuning
- alucinação → hallucination: output incorreto com aparência de correto
- embedding → embedding: representação vetorial de texto

## Recursos

- [[LLMs]]
- [[Agents]]
- [[Skills e Prompting]]

## Veja também

- [[Claude]]
- [[GitHub Copilot]]
- [[Comparativo de LLMs]]
