---
title: "Middleware pipeline"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - middleware
  - pipeline
aliases:
  - Middleware
  - Hooks
  - Onion model
---

# Middleware pipeline

> [!abstract] TL;DR
> Middleware é a pipeline de funções que processa request/response ao redor do handler. Express usa `(req, res, next)`, Fastify usa hooks nomeados, NestJS usa middleware + interceptors/guards/pipes/filters, Hono usa onion model com `await next()`.

## O que é

Pipeline é onde entram concerns transversais: logging, auth, CORS, rate limit, parsing, tracing e error handling. O conceito é comum; o modelo de cada framework muda.

## Por que importa

Quem entende só Express tende a procurar `next()` em todo lugar. Em Fastify, o ponto certo pode ser `preHandler`; em NestJS, auth pode ser Guard; em Hono, after logic vem depois de `await next()`. Saber mapear o concern para o hook certo é skill de senior.

## Como funciona

```typescript
// Express: sequencial e mutável.
app.use((req, _res, next) => {
  req.startTime = Date.now();
  next();
});
```

```typescript
// Fastify: hooks nomeados por fase.
app.addHook("onRequest", async (req) => {
  req.startTime = Date.now();
});

app.addHook("onResponse", async (req) => {
  app.log.info({ ms: Date.now() - req.startTime });
});
```

```typescript
// NestJS: interceptor com before/after.
@Injectable()
class TimingInterceptor implements NestInterceptor {
  intercept(_ctx: ExecutionContext, next: CallHandler) {
    const start = Date.now();
    return next.handle().pipe(tap(() => log(Date.now() - start)));
  }
}
```

```typescript
// Hono: onion model.
app.use("*", async (_c, next) => {
  const start = Date.now();
  await next();
  console.log(`took ${Date.now() - start}ms`);
});
```

| Framework | Modelo | Mutação | Async | Before/after |
| --- | --- | --- | --- | --- |
| Express | `next()` callback | Comum | v5 nativo | Sequencial |
| Fastify | Hooks tipados | Parcial | Sim | Fase nomeada |
| NestJS | Interceptors + hooks | Sim | Sim | Lifecycle explícito |
| Hono | Onion com `await next()` | Sim | Sim | Mesmo middleware antes/depois |

## Na prática

- Logging: Express middleware, Fastify `onResponse`, NestJS interceptor, Hono onion.
- Auth: Express middleware, Fastify `preHandler`, NestJS Guard, Hono middleware com `c.set()`.
- CORS: `cors`, `@fastify/cors`, `app.enableCors()`, middleware Hono.
- Rate limit: lib específica por framework.

## Armadilhas

1. Express: ordem de `app.use()` decide comportamento.
2. Fastify: `onRequest` roda antes de parsing/validation; nem todo dado está disponível.
3. NestJS: middleware tradicional é mais cru; interceptor/guard tem melhor integração com DI.
4. Hono: sem `await next()`, o handler posterior não executa.
5. Middleware que faz CPU-heavy work bloqueia [[Runtime e Event Loop]].

## Em entrevista

"Middleware is the cross-cutting pipeline around handlers, but each framework models it differently. Express is sequential and mutation-friendly with `(req, res, next)`. Fastify has lifecycle hooks like `onRequest`, `preHandler`, and `onResponse`. NestJS uses both traditional middleware and DI-aware hooks such as guards and interceptors. Hono uses the Koa-like onion model with `await next()`."

Vocabulário-chave:

- middleware -> middleware
- hook -> gancho
- interceptor -> interceptor
- onion model -> modelo cebola
- request lifecycle -> ciclo de vida da request

## Veja também

- [[02 - Express idiomático]]
- [[04 - NestJS - guards, interceptors, pipes, filters]]
- [[05 - Fastify - schema-first, plugins, performance]]
- [[06 - Hono e edge runtimes]]
- [[Node.js]]

