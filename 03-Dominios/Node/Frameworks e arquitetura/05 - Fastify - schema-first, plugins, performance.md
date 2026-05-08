---
title: "Fastify: schema-first, plugins, performance"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - fastify
  - schema
  - performance
aliases:
  - Fastify
  - JSON schema
  - fastify-plugin
---

# Fastify: schema-first, plugins, performance

> [!abstract] TL;DR
> Fastify é performance-focused e schema-first. Route schemas validam input e serializam output com Ajv/fast-json-stringify. Plugins são encapsulados por default. É boa escolha para APIs com contrato claro e throughput relevante.

## O que é

Fastify é um framework HTTP de baixo overhead para Node. Seus pilares são performance, developer experience, hooks, plugins/decorators e uso recomendado de JSON Schema para validation/serialization.

## Por que importa

Quando API tem contrato claro, schema-first reduz validação ad-hoc e aproxima runtime, tipos e OpenAPI. Performance não deve ser a única métrica, mas em endpoints I/O-bound com alto volume a diferença de overhead pode importar.

## Como funciona

```typescript
import Fastify from "fastify";

const app = Fastify({ logger: true });
app.get("/hello", async () => ({ greeting: "hello" }));
await app.listen({ port: 3000 });
```

```typescript
app.post(
  "/users",
  {
    schema: {
      body: {
        type: "object",
        required: ["name", "email"],
        additionalProperties: false,
        properties: {
          name: { type: "string", minLength: 1 },
          email: { type: "string", format: "email" },
        },
      },
      response: {
        201: {
          type: "object",
          required: ["id", "name", "email"],
          properties: {
            id: { type: "string" },
            name: { type: "string" },
            email: { type: "string" },
          },
        },
      },
    },
  },
  async (req, reply) => {
    const user = await db.users.create(req.body);
    return reply.code(201).send(user);
  },
);
```

```typescript
import fp from "fastify-plugin";

async function dbPlugin(fastify: FastifyInstance) {
  const db = await connectDb();
  fastify.decorate("db", db);
  fastify.addHook("onClose", async () => db.close());
}

export default fp(dbPlugin);
```

```typescript
app.addHook("onRequest", async (req) => {
  req.startTime = Date.now();
});

app.addHook("onResponse", async (req) => {
  app.log.info({ url: req.url, ms: Date.now() - req.startTime });
});
```

```typescript
import { TypeBoxTypeProvider } from "@fastify/type-provider-typebox";
import { Type } from "@sinclair/typebox";

const typed = Fastify().withTypeProvider<TypeBoxTypeProvider>();
const UserSchema = Type.Object({ name: Type.String(), email: Type.String() });

typed.post("/users", { schema: { body: UserSchema } }, async (req) => {
  return { acceptedName: req.body.name };
});
```

## Na prática

Padrão forte: schema em toda rota, plugins por feature, encapsulation como isolamento e `@fastify/swagger` para derivar OpenAPI. Use `fastify-plugin` quando um plugin precisa expor decorators ao escopo pai.

## Armadilhas

1. Schema sem `additionalProperties: false`: payloads extras passam.
2. Decorator registrado em plugin encapsulado e esperado no app pai: não vaza por design.
3. Validation async batendo em banco: pode virar DoS; use hook depois da validation.
4. Teste sem `await app.ready()` ou `await app.close()`: lifecycle incompleto.

## Em entrevista

"Fastify is schema-first and performance-focused. You declare JSON Schemas for request and response; Fastify uses Ajv for validation and fast-json-stringify for serialization. Its plugin system is encapsulated by default, which keeps feature modules isolated. It is a strong fit when your API contract is explicit and throughput matters."

Vocabulário-chave:

- schema-first -> orientado por schema
- JSON Schema -> schema JSON
- type provider -> provedor de tipos
- plugin encapsulation -> encapsulamento de plugin
- throughput -> vazão

## Fontes

- [Fastify](https://fastify.dev/)
- [Fastify validation and serialization](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/)

## Veja também

- [[01 - Os 4 frameworks - Express, NestJS, Fastify, Hono]]
- [[07 - Middleware pipeline]]
- [[09 - Validation com schema]]
- [[12 - Decision tree + cheatsheet]]
- [[Node.js]]

