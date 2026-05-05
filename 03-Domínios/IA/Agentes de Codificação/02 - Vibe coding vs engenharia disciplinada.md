---
title: "Vibe coding vs engenharia disciplinada"
created: 2026-05-02
updated: 2026-05-02
type: concept
status: seedling
publish: true
tags:
  - agentes-codificacao
  - ia
  - ferramentas
aliases:
  - Vibe coding
  - AI engineering gap
  - Disciplina de engenharia AI
---

# Vibe coding vs engenharia disciplinada

> [!abstract] TL;DR
> "Vibe coding" é gerar código por tentativa e erro conversacional com IA — funciona para protótipos mas gera tech debt exponencial em produção. Engenharia disciplinada com IA mantém o humano como arquiteto e o agente como executor, com specs claras, testes imutáveis, e code review rigoroso. O gap entre os dois é a diferença entre "funciona no meu localhost" e "funciona em produção por 3 anos". A competência profissional em 2026 está na engenharia disciplinada.

## O que é

**Vibe coding** (termo cunhado por Andrej Karpathy em 2025) descreve o workflow de:

1. Pedir algo ao AI em linguagem natural
2. Aceitar o que ele gerar
3. Se quebrar, pedir para consertar
4. Repetir até funcionar

**Engenharia disciplinada com IA** é o workflow de:

1. Definir a spec/arquitetura antes de gerar código
2. Usar o agente como executor da spec
3. Revisar cada mudança com compreensão do porquê
4. Manter testes, linting e CI como barreiras imutáveis

## Por que importa

| Métrica                | Vibe coding           | Engenharia disciplinada |
| ---------------------- | --------------------- | ----------------------- |
| Velocidade inicial     | ★★★★★                 | ★★★                     |
| Qualidade 1 mês depois | ★★                    | ★★★★★                   |
| Tech debt acumulada    | Exponencial           | Controlada              |
| Segurança              | ❌ Vulnerável          | ✅ Auditada              |
| Manutenibilidade       | Muito baixa           | Alta                    |
| Custo de tokens        | Alto (muitos retries) | Menor (menos iterações) |

## Como funciona

### O ciclo vicioso do vibe coding

```mermaid
graph TD
    A["Prompt vago<br>'faz um login'"] --> B[AI gera código]
    B --> C{Funciona?}
    C -->|Não| D["'Conserta esse erro'"]
    D --> B
    C -->|Sim| E["Commit sem review"]
    E --> F["Próximo prompt vago"]
    F --> A
    
    style E fill:#8b0000
```

**Problemas que se acumulam:**

- O AI pode "consertar" erros de forma que introduz novos bugs
- Cada iteração adiciona contexto descontrolado, aumentando custo de tokens
- Código gerado não segue padrões do projeto
- Testes são modificados pelo AI para "passar" em vez de testar corretamente

### O ciclo virtuoso da engenharia disciplinada

```mermaid
graph TD
    A["Spec clara<br>'módulo auth com JWT'"] --> B["Plan mode<br>(AI analisa, não modifica)"]
    B --> C["Review do plano"]
    C --> D["Build mode<br>(AI implementa a spec)"]
    D --> E["Code review<br>(comprehension gate)"]
    E --> F{Entende tudo?}
    F -->|Não| G["Rejeitar ou<br>pedir explicação"]
    G --> D
    F -->|Sim| H["Testes passam?"]
    H -->|Não| I["AI corrige<br>(testes imutáveis)"]
    I --> D
    H -->|Sim| J["Merge"]
    
    style J fill:#006400
```

### Práticas da engenharia disciplinada

| Prática                | O que é                                   | Por que importa                    |
| ---------------------- | ----------------------------------------- | ---------------------------------- |
| **Comprehension gate** | Se não entende a mudança, não faz merge   | Evita código fantasma no codebase  |
| **Testes imutáveis**   | AI não pode reescrever testes para passar | Testes são a barreira de qualidade |
| **Plan before build**  | Usar plan mode antes de gerar código      | Reduz iterações (e tokens)         |
| **Spec-driven**        | Definir o "o quê" antes do "como"         | Mantém coerência arquitetural      |
| **Context files**      | CLAUDE.md, .cursorrules, agents.md        | O agente segue seus padrões        |
| **Commits atômicos**   | Cada commit resolve uma coisa             | Reversibilidade granular           |

### O espectro entre vibe e disciplina

Na prática, o ideal não é nem 100% vibe nem 100% spec-driven. Depende da fase:

| Fase                                  | Abordagem recomendada                                    |
| ------------------------------------- | -------------------------------------------------------- |
| **Prototipagem / spike**              | Vibe coding é OK — descarte o código depois              |
| **MVP v1**                            | Semi-disciplinado — specs leves, testes básicos          |
| **Feature em produção**               | Disciplinado — specs, testes, review                     |
| **Infraestrutura / auth / pagamento** | Ultra-disciplinado — human review obrigatório, zero vibe |

## Armadilhas

- **"Vibe coding é ruim"** — não é. É excelente para prototipagem, exploração, e aprendizado. O problema é usá-lo em produção.
- **"Engenharia disciplinada é lenta"** — parece mais lenta no primeiro dia. No dia 30, o projeto com disciplina está muito à frente porque não está gastando tempo consertando tech debt.
- **"O AI é tão bom que review não precisa"** — falso. LLMs alucinam, ignoram edge cases, e introduzem vulnerabilidades silenciosas. Review é obrigatório.
- **"Testes são opcionais com AI"** — ao contrário: com AI, testes são MAIS importantes. Sem testes, não há barreira entre código correto e alucinação que funciona por acaso.
- **Reescrever testes para passar** — se o agente modifica os testes junto com o código, os testes não estão testando nada. Testes devem ser escritos ANTES ou separadamente.

## Veja também

- [[01 - De autocomplete a agentes autônomos]] — a evolução que levou ao gap
- [[03 - O comprehension gate]] — a prática central da disciplina
- [[14 - agents.md e configuração de projeto]] — como configurar o agente para trabalhar com disciplina

## Referências

- **Karpathy, Andrej** — *Vibe Coding* (X/Twitter, 2025). O termo original e seu contexto.
- **Plus8Soft** — *The Comprehension Gate* (2026). Framework de code review para AI.
- **Eventuallymaking.io** — *AI-Assisted Engineering vs Vibe Coding* (2026). Análise do gap.
