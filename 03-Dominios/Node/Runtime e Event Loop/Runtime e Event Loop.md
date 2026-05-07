---
title: "Runtime e Event Loop"
created: 2026-05-07
updated: 2026-05-07
type: moc
status: seedling
publish: true
tags:
  - node
  - event-loop
  - moc
aliases:
  - Event Loop
  - Node Runtime
  - Galho 1 - Runtime
---

# Runtime e Event Loop

> [!abstract] TL;DR
> Galho 1 da trilha Node Senior. Cobre o "motor" do Node — single-thread, V8, libuv, fases do event loop, microtasks/macrotasks, promises por dentro, async/await desmistificado, bloqueio do loop e diagnóstico. Pré-requisito de todos os outros galhos (paralelismo, streams, frameworks, observability, segurança).

## Sobre este galho

(introdução curta — preencher na Task 14)

## Comece por aqui — trilha completa (12 notas)

### Bloco A — Mental model

1. [[01 - Single-thread e non-blocking I-O]]
2. [[02 - V8, libuv e thread pool]]
3. [[03 - Call stack, heap e queues]]

### Bloco B — Event loop deep dive

4. [[04 - As fases do event loop]]
5. [[05 - Microtasks - nextTick, queueMicrotask, Promise.then]]
6. [[06 - Macrotasks e timers - setTimeout, setInterval, setImmediate]]
7. [[07 - I-O assíncrono - kernel vs thread pool]]

### Bloco C — async/await em profundidade

8. [[08 - Promises por dentro]]
9. [[09 - async-await - o que é, o que não é]]

### Bloco D — Bloqueio e diagnóstico

10. [[10 - Bloqueio do event loop - sintomas e causas]]
11. [[11 - Diagnóstico do event loop]]

### Bloco E — Fechamento

12. [[12 - Armadilhas, regras práticas, cheatsheet]]

## Rotas alternativas

### Rota entrevista internacional

01 → 03 → 04 → 05 → 06 → 09 → 10. Foco em "explicar o motor pra entrevistador".

### Rota debugging em produção

01 → 04 → 07 → 10 → 11. Foco em "minha app tá lenta, por quê?".

### Rota async/await

03 → 05 → 08 → 09. Foco em entender o "porquê" das promises.

## Todas as notas

```dataview
TABLE status, updated
FROM "03-Dominios/Node/Runtime e Event Loop"
WHERE type = "concept"
SORT file.name ASC
```

## Veja também

- [[03-Dominios/Node/index|Node.js (MOC central)]]
- [[Node.js]] — tronco (deep dive panorâmico)
- [[JavaScript Fundamentals]] — event loop básico do JS
