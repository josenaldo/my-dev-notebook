---
title: "STAR Method"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - entrevista
  - behavioral
publish: false
---

# STAR Method

Framework para estruturar respostas em entrevistas behavioral — Situation, Task, Action, Result.

## O que é

STAR é um framework para contar histórias profissionais de forma estruturada e impactante. Cada resposta tem 4 partes que guiam o entrevistador pela sua experiência.

## Como funciona

### S — Situation (Contexto)

Onde e quando? Qual era o cenário? Mantenha breve (2-3 frases).

> "At Muvz, I joined a project to modernize a major newspaper's legacy system. The existing architecture was a Java EJB monolith, and the project was already 3 months behind schedule."

### T — Task (Desafio)

O que precisava ser feito? Qual era o seu papel específico?

> "My task was to lead the migration from the monolith to a microservices architecture using Spring Boot and Kafka, while mentoring the team on modern practices."

### A — Action (O que você fez)

O que VOCÊ fez? Seja específico sobre suas ações, decisões e por que.

> "I started by mapping the bounded contexts using Event Storming to identify natural service boundaries. Then I led the team in building 5 Spring Boot microservices, introduced Kafka for event-driven communication between services, and implemented Hexagonal Architecture to keep the business logic independent from infrastructure."

### R — Result (Resultado)

Qual foi o impacto? Use números sempre que possível.

> "We improved system performance by over 40%, eliminated the 3-month delay, and delivered on time. The team adopted DDD and Hexagonal Architecture as standard practices for future projects."

## Banco de histórias STAR

### Liderança técnica — Kafka na Muvz

- **S:** Projeto de modernização de sistema legado para um grande jornal. Monolito Java EJB.
- **T:** Liderar migração para microserviços e introduzir mensageria assíncrona.
- **A:** Event Storming para mapear bounded contexts. 5 microserviços Spring Boot. Proof-of-concept com Kafka. Setup do cluster Kafka em Kubernetes com DevOps. Mentoria da equipe.
- **R:** Performance +40%. 3 meses de atraso eliminados. Equipe adotou DDD e Hexagonal como padrão.

### CI/CD — MedEspecialista

- **S:** Plataforma de educação médica com deploys manuais e lentos.
- **T:** Automatizar pipeline de entrega e modernizar a stack frontend.
- **A:** GitHub Actions para CI/CD completo. Migração de CRA para Vite + TypeScript. TanStack Query para data fetching. BullMQ para processamento assíncrono.
- **R:** Deploy de 1 hora → 2 minutos. Trabalho manual reduzido em 97%. Lead time de 2 semanas → 1 semana.

### Mentoria — Arquitetura na Muvz

- **S:** Equipe de desenvolvedores trabalhando sem padrões consistentes.
- **T:** Elevar o nível técnico da equipe mantendo produtividade.
- **A:** Code reviews focados em princípios (SOLID, DDD). Sessões de pair programming. Documentação de decisões arquiteturais. Exemplos práticos no código.
- **R:** Equipe internalizou práticas de arquitetura limpa. Menos bugs. Código mais consistente.

## Armadilhas comuns

- **Situation muito longa:** máximo 2-3 frases. O entrevistador quer saber o que VOCÊ fez.
- **Action vaga:** "eu trabalhei com a equipe para resolver" — especifique SUA contribuição.
- **Sem números no Result:** "ficou melhor" vs "melhorou 40%". Números são mais impactantes.
- **Esquecer o aprendizado:** para histórias de fracasso, adicione o que aprendeu (STAR + L).

## How to explain in English

"I structure my behavioral answers using STAR. Let me give you an example:

**Situation:** At Muvz, I joined a project to modernize a major newspaper's legacy Java EJB system. The project was already three months behind schedule.

**Task:** I was responsible for leading the migration to a microservices architecture and getting the project back on track.

**Action:** I started with Event Storming to identify bounded contexts and natural service boundaries. Then I built a Kafka proof-of-concept to demonstrate event-driven communication. We split the monolith into five Spring Boot microservices following Hexagonal Architecture. I also mentored the team on DDD and SOLID principles through code reviews and pair programming.

**Result:** We improved performance by over 40%, eliminated the three-month delay, and delivered on time. The team adopted these architectural practices as their standard going forward."

### Key vocabulary

- situação → situation / context
- tarefa → task / challenge
- ação → action: o que você fez
- resultado → result / outcome / impact
- impacto mensurável → measurable impact
- lições aprendidas → lessons learned

## Veja também

- [[Behavioral Questions]]
- [[Communicating Trade-offs]]
- [[Minha Narrativa Profissional]]
