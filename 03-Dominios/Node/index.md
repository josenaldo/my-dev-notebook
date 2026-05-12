---
title: "Node.js"
type: moc
publish: true
created: 2026-05-04
updated: 2026-05-09
status: seedling
progresso: pendente
tags:
  - node
  - moc
  - backend
aliases:
  - NodeJS
  - Node
---

# Node.js

Estante de Node.js: runtime, event loop, módulos, frameworks de servidor (Express, NestJS), ORMs (Sequelize, Prisma), padrões arquiteturais (Clean Architecture), tooling do ecossistema, e práticas de backend com JavaScript/TypeScript.

## Conteúdo

### Galhos da trilha Node Senior

- [[Runtime e Event Loop]] — galho 1: o motor do Node (single-thread, libuv, fases, microtasks, async/await, bloqueio, diagnóstico)
- [[Paralelismo]] — galho 2: as 3 ferramentas de paralelismo (Worker Threads, Cluster, child_process), SharedArrayBuffer/Atomics, pool de workers, contexto de produção, decision tree
- [[Streams]] — galho 3: abstração fundamental para processar dados em chunks (4 tipos, backpressure, pipeline, async iter, Web Streams, padrões práticos, performance)
- [[Frameworks e arquitetura]] — galho 4: os 4 frameworks principais (Express, NestJS, Fastify, Hono), patterns transversais (middleware, error handling, validation), Clean Architecture e DI
- [[Observability e produção]] — galho 5: logs, métricas, traces, profiling avançado, SLOs, dashboards Grafana, alertas multi-janela e checklists de produção
- [[ORMs e banco de dados]] — galho 6: os 4 ORMs principais (Sequelize, Prisma, TypeORM, Drizzle), padrões críticos de N+1, migrations, transações e paginação, decision tree para escolha de ORM

### Outras notas

- [[Ferramentas Node]] — panorama de ferramentas do ecossistema (em construção)

## Veja também

- [[03-Domínios/JavaScript/index|JavaScript]]
- [[03-Dominios/TypeScript/index|TypeScript]]
- [[Ferramentas|Ferramentas]]
