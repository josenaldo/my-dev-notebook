---
title: "Decision tree + cheatsheet"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - cheatsheet
  - decision-tree
  - referencia
aliases:
  - Cheatsheet frameworks
  - Decision tree frameworks
---

# Decision tree + cheatsheet

> [!abstract] TL;DR
> Fechamento do galho: decision tree para escolher framework, tabela comparativa, top armadilhas e vocabulário PT->EN. Use como revisão rápida antes de entrevista ou decisão de arquitetura.

## Decision tree - qual framework para qual contexto

```text
Qual o contexto?

├─ Microsserviço I/O-bound simples, time pequeno
│  ├─ Performance/contrato importam muito? -> Fastify
│  └─ Time conhece Express e quer simplicidade? -> Express
│
├─ App enterprise, domínio complexo, time grande
│  ├─ DI, módulos e convenção corporativa importam? -> NestJS
│  └─ Time quer controle manual? -> Express/Fastify + Clean Architecture
│
├─ API com schema bem definido + throughput alto
│  └─ -> Fastify
│
├─ Edge worker / serverless multi-runtime
│  └─ -> Hono
│
└─ Migração de legacy
   ├─ Express legacy saudável -> modernizar Express
   └─ Framework abandonado -> avaliar Fastify ou NestJS
```

## Tabela comparativa

| Atributo | Express | NestJS | Fastify | Hono |
| --- | --- | --- | --- | --- |
| Modelo | Middleware-based | Opinionated, DI | Schema-first | Edge-first |
| DI built-in | Não | Sim | Não | Não |
| Validation | Manual/zod/joi | `ValidationPipe` ou zod | JSON Schema nativo | zod via lib |
| Performance | Baseline | Adapter-dependent | Baixo overhead | Edge-optimized |
| Maturidade | Alta | Alta | Alta | Crescente |
| Ecossistema | Maior | Médio/Nest-specific | Médio | Menor, crescendo |
| Learning curve | Baixa | Alta | Média | Baixa |
| Use case | Glue, protótipo, microsserviço | Enterprise | APIs throughput-focused | Edge/serverless |

## Top armadilhas

1. Express 4 sem `asyncHandler`: erro async não capturado.
2. Express error middleware com 3 args: precisa de `(err, req, res, next)`.
3. Mutar `req` sem contrato: ordem de middleware vira bug invisível.
4. NestJS `Scope.REQUEST` sem necessidade: custo propaga.
5. NestJS circular import resolvido com `forwardRef()` sem refatorar design.
6. Fastify schema sem `additionalProperties: false`.
7. Fastify plugin encapsulado quando deveria expor decorator ao pai.
8. Hono assumindo APIs Node em edge runtime.
9. Hono middleware sem `await next()`.
10. Stack trace em produção.
11. DI container em app pequeno sem necessidade.
12. Clean Architecture em CRUD simples.
13. Schema só para body, ignorando query/params/headers.
14. Misturar zod e `class-validator` sem convenção.

## Vocabulário PT->EN

| PT | EN |
| --- | --- |
| middleware | middleware |
| middleware de erro | error middleware |
| decorador | decorator |
| injeção de dependência | dependency injection |
| provedor | provider |
| módulo | module |
| controlador | controller |
| escopo | scope |
| guarda | guard |
| interceptor | interceptor |
| pipe | pipe |
| filtro | filter |
| ciclo de vida | lifecycle |
| orientado por schema | schema-first |
| inferência de tipo | type inference |
| encapsulamento de plugin | plugin encapsulation |
| runtime de borda | edge runtime |
| nativo da Fetch API | Fetch API native |
| modelo cebola | onion model |
| taxonomia de erros | error taxonomy |
| raiz de composição | composition root |
| portas e adaptadores | ports and adapters |
| arquitetura hexagonal | hexagonal architecture |
| inversão de dependência | dependency inversion |

## Próximos galhos

- Observability: logging estruturado, métricas, tracing, profiling e diagnóstico de frameworks em produção.
- Segurança: Helmet, CORS, rate limiting, CSRF, input sanitization e hardening por framework.
- Base técnica: [[Runtime e Event Loop]], [[Paralelismo]] e [[Streams]].

## Em entrevista

"I choose a Node framework by matching the problem shape. Express is simple and ubiquitous, Fastify is schema-first and low-overhead, NestJS is the structured enterprise choice with dependency injection, and Hono is the edge-first multi-runtime option. I would not rank them globally; I would choose based on deploy target, API contract, team size, and domain complexity."

Vocabulário-chave:

- decision tree -> árvore de decisão
- trade-off -> troca/custo-benefício
- problem shape -> formato do problema
- deploy target -> alvo de deploy
- framework fit -> encaixe do framework

## Veja também

- [[Frameworks e arquitetura]]
- [[01 - Os 4 frameworks - Express, NestJS, Fastify, Hono]]
- [[02 - Express idiomático]]
- [[03 - NestJS - fundamentos]]
- [[04 - NestJS - guards, interceptors, pipes, filters]]
- [[05 - Fastify - schema-first, plugins, performance]]
- [[06 - Hono e edge runtimes]]
- [[07 - Middleware pipeline]]
- [[08 - Error handling estruturado]]
- [[09 - Validation com schema]]
- [[10 - Clean Architecture em Node]]
- [[11 - DI - manual vs container]]
- [[Node.js]]
- [[Runtime e Event Loop]]
- [[Paralelismo]]
- [[Streams]]

