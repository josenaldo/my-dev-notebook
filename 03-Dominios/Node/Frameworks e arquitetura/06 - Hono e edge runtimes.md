---
title: "Hono e edge runtimes"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - hono
  - edge
  - cloudflare-workers
  - serverless
aliases:
  - Hono
  - edge runtime
  - Cloudflare Workers
  - Deno Deploy
---

# Hono e edge runtimes

> [!abstract] TL;DR
> Hono é um framework ultralight baseado em Web Standards e Fetch API. Roda em Cloudflare Workers, Deno, Bun, AWS Lambda e Node. É boa escolha quando o deploy é edge/serverless multi-runtime; é ruim quando o app depende de APIs Node-only.

## O que é

Hono é um framework HTTP minimalista e multi-runtime. Em vez de `req`/`res` estilo Node, ele trabalha perto de `Request`/`Response` da Fetch API e expõe um contexto `c`.

## Por que importa

Edge runtimes mudam o contrato. Cloudflare Workers e similares não são "Node completo em outro lugar": `fs`, TCP custom, processos longos e várias libs nativas podem não existir. Hono dá uma API uniforme para escrever Web APIs nesse ambiente.

## Como funciona

```typescript
import { Hono } from "hono";

const app = new Hono();
app.get("/hello", (c) => c.json({ greeting: "hello" }));

export default app;
```

```typescript
// Mesmo app.fetch pode ser adaptado por runtime.
// Cloudflare Workers: export default app;
// Node: serve({ fetch: app.fetch })
// Deno: Deno.serve(app.fetch)
// Bun: export default { fetch: app.fetch }
```

```typescript
app.use("*", async (c, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  console.log(`${c.req.method} ${c.req.url} ${ms}ms`);
});
```

```typescript
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";

const CreateUser = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

app.post("/users", zValidator("json", CreateUser), (c) => {
  const data = c.req.valid("json");
  return c.json({ id: crypto.randomUUID(), ...data }, 201);
});
```

```typescript
import { HTTPException } from "hono/http-exception";

app.onError((err, c) => {
  const status = err instanceof HTTPException ? err.status : 500;
  return c.json(
    { type: "about:blank", title: "Error", status, detail: err.message },
    status,
    { "Content-Type": "application/problem+json" },
  );
});
```

## Na prática

Use Hono quando o deploy é Cloudflare Workers, Vercel Edge, Deno Deploy, Lambda@Edge ou quando a portabilidade multi-runtime é requisito. Prefira Express/Fastify/NestJS quando o app é Node-only, precisa de libs Node-specific ou tem integração profunda com filesystem, sockets, streams Node ou infraestrutura tradicional.

## Armadilhas

1. Assumir `fs`/`net` em edge runtime: falha fora de Node.
2. Fazer CPU-heavy handler em edge: limites de tempo e CPU são menores.
3. Guardar estado mutável global: edge pode escalar/isolar instâncias de forma diferente.
4. Esquecer `await next()` no middleware onion: o handler posterior não roda.

## Em entrevista

"Hono is an ultralight, multi-runtime framework built on Web Standards. It uses the Fetch API model instead of Node's `req` and `res`, and it runs on Cloudflare Workers, Deno, Bun, AWS Lambda, and Node. The decision is deploy-driven: choose it for edge or serverless multi-runtime apps; avoid it when the application depends deeply on Node-only APIs."

Vocabulário-chave:

- edge runtime -> runtime de borda
- Fetch API native -> nativo da Fetch API
- multi-runtime -> múltiplos runtimes
- ultralight -> ultraleve
- onion middleware -> middleware em cebola

## Fontes

- [Hono docs](https://hono.dev/docs)

## Veja também

- [[01 - Os 4 frameworks - Express, NestJS, Fastify, Hono]]
- [[07 - Middleware pipeline]]
- [[09 - Validation com schema]]
- [[12 - Decision tree + cheatsheet]]
- [[Node.js]]

