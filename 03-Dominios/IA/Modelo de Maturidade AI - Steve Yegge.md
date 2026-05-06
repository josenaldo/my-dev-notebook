---
title: "Modelo de Maturidade AI - Steve Yegge"
created: 2026-05-05
updated: 2026-05-05
type: concept
status: seedling
progresso: andamento
tags:
  - IA
  - EngenhariaDeSoftware
  - Carreira
  - Produtividade
publish: true
---

# Modelo de Maturidade AI - Steve Yegge

O modelo de maturidade de Steve Yegge (ex-Google, ex-Amazon, atual Head de Engenharia na Sourcegraph) descreve a evolução da relação entre o engenheiro de software e a Inteligência Artificial. Ele mapeia o caminho desde a resistência inicial até a orquestração completa de sistemas autônomos.

## O que é

É um framework de 8 níveis (0 a 7) que define a transição do desenvolvedor de um **trabalhador manual de sintaxe** para um **orquestrador de arquitetura**. O modelo postula que a IA não substitui o especialista, mas o amplifica, exigindo maior (e não menor) conhecimento técnico conforme se sobe de nível.

## Os 8 Níveis da Ascensão AI

O modelo é dividido em três fases principais:

### Fase 1: O Usuário de Ferramenta (Níveis 0 - 2)

Aqui, o humano é o único "pensador". A IA é um utilitário menor.

- **Nível 0 (O Negacionista):** "IA é apenas hype". Escreve 100% manual. Risco alto de obsolescência rápida.
- **Nível 1 (O Cauteloso):** "O Autocomplete". Usa sugestões de linha única no IDE (Copilot padrão).
- **Nível 2 (O Questionador):** "Substituto do Stack Overflow". Usa chat externo para perguntas contextuais, mas implementa tudo manualmente.

> [!important] **A Grande Fenda (The Great Chasm)**
> Entre o Nível 2 e o 3 existe um abismo mental. Para cruzá-lo, você deve parar de ser o "escritor" e começar a ser o "delegador". Isso exige confiança na execução da IA e contexto profundo do projeto.

### Fase 2: O Salto para a Autonomia (Níveis 3 - 4)

Onde ocorre o ganho real de alavancagem (Leverage).

- **Nível 3 (O Delegador Básico):** Começa a pedir código, não apenas respostas. Ainda micromanageia (pede funções pequenas, copia e cola).
- **Nível 4 (O Diretor):** Mudança de modelo mental. Você especifica **comportamento** (via testes ou specs) e a IA implementa o bloco lógico inteiro. O trabalho é validar resultados, não conferir sintaxe.

### Fase 3: O Gerente e Arquiteto AI (Níveis 5 - 7)

A era da liderança de agentes digitais.

- **Nível 5 (O Orquestrador):** Usa ferramentas agentinas (Agent Mode). A IA tem autonomia para navegar no codebase, ler arquivos, rodar testes e se auto-corrigir.
- **Nível 6 (O Gerente de Multi-Agentes):** Coordena múltiplos agentes em paralelo em tarefas diferentes. O dev atua como um Tech Lead de robôs.
- **Nível 7 (O Arquiteto):** Foco total em design de sistema, contratos de API e critérios de qualidade. Os agentes constroem a estrutura baseada nesses designs.

## Como saltar de nível

1. **De 2 para 3:** Pare de perguntar "como fazer X" e passe a comandar "escreva o código para X". Delegue tarefas isoladas completas (ex: "escreva a suite de testes para este componente").
2. **De 3 para 4:** Adote **Specification-First**. Escreva o teste ou o arquivo de requisitos primeiro. Deixe a IA implementar e avalie o resultado contra o que você definiu.
3. **De 4 para 5:** Use ferramentas agentinas (como `agent mode` do Claude Code, Cody ou Cursor) por uma semana inteira em um projeto real. Force-se a não editar o código manualmente; dê instruções e deixe o agente agir.

## Por que é importante

O baseline de produtividade da indústria está subindo. Desenvolvedores em níveis altos (5+) operam com uma alavancagem até **5x maior** que os de nível 2. Permanecer nos níveis baixos enquanto a indústria avança não é ficar parado — é tornar-se obsoleto.

## Pontos Essenciais para o Desenvolvedor

- **Expertise Inversa:** Conforme você delega mais, a necessidade de conhecimento sênior **aumenta**. É preciso saber o que é "bom" para validar o que a IA entrega.
- **Vibe Coding:** Termo cunhado por Andrej Karpathy que ressoa com este modelo — a habilidade de "guiar" a IA através do contexto e intenção, em vez de comandos manuais.
- **Testes como Especificação:** No Nível 4+, os testes deixam de ser um "extra" e passam a ser a ferramenta principal de programação.

## Aprofundamento

> [!convite] Referências do Modelo
>
> - [Welcome to Gas Town](https://sourcegraph.com/blog/welcome-to-gas-town) — Blog da Sourcegraph por Steve Yegge.
> - [The Death of the Junior Developer](https://steve-yegge.medium.com/) — Discussões sobre o impacto da IA na carreira.

## Veja também

- [[Context Engineering]]
- [[Claude Code - Cron job para iniciar janela de 5h]]
- [[Minha Narrativa Profissional]]
