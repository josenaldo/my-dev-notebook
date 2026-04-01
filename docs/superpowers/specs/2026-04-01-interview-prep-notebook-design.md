# Interview Prep Notebook — Design Spec

## Context

Josenaldo é um Senior Fullstack Developer com 20+ anos de experiência (Java/Spring, Node/TS, React). Ele tem um vault Obsidian (`my-dev-notebook`) organizado por domínio com MOCs, mas as notas são majoritariamente coleções de links externos — falta profundidade de conteúdo.

O objetivo é transformar esse vault em material de estudo para entrevistas técnicas em vagas internacionais remotas (foco imediato) e Senior/Staff em startups (futuro). O conteúdo deve ser bilíngue (PT-BR + inglês técnico) e cobrir ambas as stacks principais: Java/Spring e JS/TS/React/Node.

## Abordagem escolhida: Híbrida

Enriquecer as notas existentes nos domínios com conteúdo bilíngue profundo + criar uma Trilha de Entrevistas em `Aprendizado/` como roteiro sequencial com links cruzados.

---

## 1. Estrutura de pastas

### Organização por stack

```
my-dev-notebook/
├── _guia/                              ← NOVO
│   └── Como usar este vault.md
├── _templates/
│   └── Template - Interview Note.md    ← NOVO
│
├── Fundamentos/                        ← ENRIQUECIDO (agnóstico)
│   ├── Algoritmos.md
│   ├── Estruturas de Dados.md          ← novo
│   ├── Banco de dados.md
│   ├── Orientação a Objetos.md
│   ├── Testes.md
│   ├── Redes e Protocolos.md           ← novo
│   └── Sistemas Operacionais.md        ← novo
│
├── Arquitetura/                        ← ENRIQUECIDO (agnóstico)
│   ├── System Design.md               ← novo
│   ├── Design Patterns.md             ← novo
│   ├── API Design.md                  ← novo
│   └── ... (existentes expandidos)
│
├── Java/                               ← STACK: reorganizado
│   ├── Core/
│   │   └── Java Fundamentals.md
│   ├── Backend/
│   │   ├── Spring Boot.md
│   │   └── Kafka/ (existente)
│   └── ... (existentes)
│
├── JavaScript/                         ← STACK: novo (absorve Frontend/)
│   ├── Core/
│   │   ├── JavaScript Fundamentals.md
│   │   └── TypeScript.md
│   ├── Backend/
│   │   └── Node.js.md
│   └── Frontend/
│       └── React.md
│
├── Python/                             ← STACK: futuro
├── Go/                                 ← STACK: futuro
│
├── Aprendizado/
│   ├── Trilha Entrevistas.md           ← NOVO
│   └── ... (existentes mantidos)
│
├── Infraestrutura/                     ← existente, sem mudanças
├── Ferramentas/                        ← existente, sem mudanças
├── Recursos/                           ← existente, sem mudanças
└── MOCs/
    ├── MOC - JavaScript.md             ← novo
    └── ... (existentes mantidos)
```

### Princípios da organização

- **Fundamentos + Arquitetura** = conhecimento agnóstico de stack
- **Java/, JavaScript/, Python/, Go/** = cada stack é um "mundo" com `Core/`, `Backend/` e `Frontend/` quando cabível
- **JavaScript/** absorve TypeScript e web (React) — mesmo ecossistema
- **_guia/** contém documentação sobre como usar o vault e a trilha de estudo
- Pastas Python/ e Go/ ficam como placeholders para importação futura

---

## 2. Template bilíngue — Interview Note

Extensão do Template - Nota.md existente, com seções adicionais para entrevistas.

```yaml
---
title: "<% tp.file.title %>"
created: <% tp.date.now("YYYY-MM-DD") %>
updated: <% tp.date.now("YYYY-MM-DD") %>
type: concept
status: seedling
tags: []
publish: false
---

# [Título]

<!-- Uma frase sobre o conceito -->

## O que é
<!-- Definição clara em PT-BR -->

## Como funciona
<!-- Mecanismo, funcionamento -->

## Quando usar
<!-- Casos de uso, contexto -->

## Armadilhas comuns
<!-- O que evitar -->

## Na prática (da minha experiência)
<!-- Exemplo real de quando usou isso em projetos -->

## How to explain in English
<!-- 2-3 parágrafos de como articular esse conceito
     em uma entrevista em inglês, com vocabulário chave -->

### Key vocabulary
- termo_pt → english_term: definição curta

## Recursos
- links úteis

## Veja também
- [[notas relacionadas]]
```

---

## 3. Trilha de Entrevistas

Arquivo `Aprendizado/Trilha Entrevistas.md` — roteiro sequencial com checkboxes para tracking.

### Fases

**Fase 1 — Fundamentos (base sólida)**
O que qualquer senior precisa articular com clareza.

- Estruturas de Dados
- Algoritmos
- Orientação a Objetos
- Design Patterns
- Banco de dados
- Testes
- Redes e Protocolos

**Fase 2 — Arquitetura & System Design**
O diferencial de senior: tomar decisões e justificar.

- System Design
- API Design
- Arquitetura de Software (existente)
- Event Storming (existente)

**Fase 3 — Stack Java**
Profundidade no backend principal.

- Java Fundamentals
- Spring Boot
- Kafka (existente)

**Fase 4 — Stack JavaScript/TypeScript**
Fullstack: dominar o ecossistema JS.

- JavaScript Fundamentals
- TypeScript
- Node.js
- React

**Fase 5 — Behavioral & Comunicação**
Como se vender em inglês e demonstrar senioridade.

- Behavioral Questions
- STAR Method
- Communicating Trade-offs
- Minha Narrativa Profissional

**Fase 6 — Prática**
Mock interviews e exercícios.

- System Design Practice
- Coding Challenges Strategy

---

## 4. Padrão de profundidade por tipo de nota

| Tipo de nota | Profundidade | Exemplo |
|---|---|---|
| Conceito core | 800-1500 palavras | Estruturas de Dados, OOP, System Design |
| Ferramenta/framework | 500-800 palavras | Spring Boot, React, Node.js |
| Behavioral | 300-500 palavras + exemplos STAR | Minha Narrativa, STAR Method |
| Prática | Checklists + links | Coding Challenges Strategy |

---

## 5. Idioma

- **Conteúdo principal:** PT-BR
- **Termos técnicos:** sempre em inglês
- **Seção "How to explain in English":** em cada nota, com vocabulário-chave
- **Key vocabulary:** glossário bilíngue no final de cada seção EN

---

## 6. O que fazer com conteúdo existente

- **Notas com links:** manter os links na seção "Recursos", adicionar conteúdo explicativo acima
- **Trilhas existentes (Trilha Java, Trilha Backend, etc.):** manter como estão — a Trilha de Entrevistas faz referência cruzada
- **MOCs existentes:** adicionar links para novas notas quando relevante
- **Frontend/ (pasta existente vazia):** será substituída por JavaScript/Frontend/

---

## 7. _guia/ — Como usar este vault

Uma pasta com documentação sobre:
- Como o vault está organizado
- Como usar a Trilha de Entrevistas (seguir as fases, marcar checkboxes)
- Como criar novas notas (qual template usar)
- Como usar as seções bilíngues para praticar inglês técnico

---

## 8. Verificação

Para testar que o design funciona:

1. Criar a estrutura de pastas
2. Criar o template Interview Note
3. Criar a Trilha de Entrevistas com links
4. Enriquecer 2-3 notas piloto (uma de Fundamentos, uma de stack)
5. Criar o _guia/
6. Verificar que os links [[wiki]] resolvem corretamente no Obsidian
7. Verificar que os MOCs com dataview listam as notas novas
8. Abrir o vault no Obsidian e navegar pela trilha
