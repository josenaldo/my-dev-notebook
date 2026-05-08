---
title: "Express idiomático"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - express
  - middleware
aliases:
  - Express
  - asyncHandler
  - error middleware
---

# Express idiomático

> [!abstract] TL;DR
> Express é middleware-based: uma pipeline ordenada de funções `(req, res, next)`. Express 5 encaminha rejeições de Promises para `next(err)` automaticamente; Express 4 ainda exige wrapper em muito código legacy. Error middleware tem quatro argumentos: `(err, req, res, next)`.

## O que é

Express é o framework HTTP minimalista mais conhecido do ecossistema Node. Ele oferece routing, middleware e integração direta com `http`, mas deixa validação, DI, OpenAPI, auth e organização arquitetural por conta da aplicação.

## Por que importa

Express continua aparecendo em entrevistas e projetos reais porque é simples, estável e bem conhecido. Código Express idiomático em 2026 é diferente de código Express 4 escrito sem TypeScript, sem schema e sem erro global consistente.

## Como funciona

```typescript
import express from "express";

const app = express();
app.use(express.json({ limit: "1mb" }));

app.get("/hello", (_req, res) => {
  res.json({ greeting: "hello" });
});

app.listen(3000);
```

```typescript
// Express 5: reject/throw em async handler chega ao error middleware.
app.get("/users/:id", async (req, res) => {
  const user = await db.users.findById(req.params.id);
  if (!user) throw new NotFoundError("User not found");
  res.json(user);
});
```

```typescript
// Express 4 ou compat: wrapper ainda aparece em codebases legacy.
const asyncHandler =
  <T extends express.RequestHandler>(fn: T): express.RequestHandler =>
  (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };

app.get(
  "/users/:id",
  asyncHandler(async (req, res) => {
    const user = await db.users.findById(req.params.id);
    res.json(user);
  }),
);
```

```typescript
import type { NextFunction, Request, Response } from "express";

app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  if (res.headersSent) return next(err);

  const status = err instanceof HttpError ? err.status : 500;
  res.status(status).type("application/problem+json").json({
    type: "about:blank",
    title: err.name,
    status,
    detail: status >= 500 ? "Unexpected error" : err.message,
    instance: req.originalUrl,
  });
});
```

```typescript
const userRouter = express.Router();
userRouter.get("/", listUsers);
userRouter.get("/:id", getUser);
userRouter.post("/", createUser);

app.use("/api/v1/users", userRouter);
```

## Na prática

Padrão típico em projetos TypeScript novos: Express 5 + `zod` + error middleware global + middlewares explícitos (`helmet`, `cors`, logger, rate limit) + routers por feature. `express.json({ limit })` deve ter limite explícito; body sem limite é porta para abuso de memória.

## Armadilhas

1. Express 4 com handler async sem wrapper: rejeição não chega ao error middleware.
2. Error middleware com 3 argumentos: Express não o reconhece como handler de erro.
3. Mutar `req` em middleware sem tipo/documentação: a ordem vira contrato invisível.
4. Chamar `res.send()` e depois `next(err)`: risco de `Cannot set headers after they are sent`.
5. Registrar error middleware antes das rotas: ele não captura o que vem depois.

## Em entrevista

"Express is a minimal middleware-based framework. In Express 5, promise rejections from async route handlers automatically call `next(err)`, while Express 4 codebases commonly need an `asyncHandler` wrapper. Error middleware is different from regular middleware: it has four arguments, with `err` first, and it should be registered after routes."

Vocabulário-chave:

- middleware pipeline -> pipeline de middleware
- error middleware -> middleware de erro
- async wrapper -> wrapper assíncrono
- router mounting -> montagem de routers
- body limit -> limite de payload

## Fontes

- [Express](https://expressjs.com/)
- [Express error handling](https://expressjs.com/en/guide/error-handling.html)

## Veja também

- [[01 - Os 4 frameworks - Express, NestJS, Fastify, Hono]]
- [[07 - Middleware pipeline]]
- [[08 - Error handling estruturado]]
- [[09 - Validation com schema]]
- [[Node.js]]

