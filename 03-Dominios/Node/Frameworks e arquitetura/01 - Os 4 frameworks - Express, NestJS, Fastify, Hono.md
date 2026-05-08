---
title: "Os 4 frameworks: Express, NestJS, Fastify, Hono"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - mental-model
  - express
  - nestjs
  - fastify
  - hono
aliases:
  - Visão geral frameworks Node
  - 4 frameworks Node
---

# Os 4 frameworks: Express, NestJS, Fastify, Hono

> [!abstract] TL;DR
> Em 2026, a decisão pragmática em Node passa por quatro modelos: Express (middleware-based e ubíquo), NestJS (opinativo, DI e decorators), Fastify (schema-first e performance), Hono (edge-first e Fetch API). Não existe campeão universal. Existe matching problema -> ferramenta.

## O que é

**Express** é o framework minimalista e "unopinionated" do ecossistema Node. A documentação oficial o descreve como uma camada fina de features web com uso forte de middleware; em 2026, Express 5.x é a linha corrente.

**NestJS** é um framework TypeScript opinativo para server-side apps escaláveis. O modelo central combina módulos, controllers, providers, decorators e um container de dependency injection.

**Fastify** é um framework HTTP focado em baixo overhead, plugin architecture e schemas. A própria documentação recomenda JSON Schema para validar rotas e serializar respostas.

**Hono** é um framework pequeno e multi-runtime baseado em Web Standards. Roda em Cloudflare Workers, Deno, Bun, AWS Lambda e Node, com API baseada em `Request`/`Response`.

## Por que importa

Confundir os modelos leva a escolhas caras. Usar NestJS em um microserviço simples pode introduzir DI e decorators sem retorno. Usar Express por hábito em um domínio enterprise grande pode deixar estrutura demais na disciplina individual de cada dev. Usar Fastify sem schema desperdiça o diferencial do framework. Usar Hono em app que depende profundamente de `fs`, `net` ou bibliotecas Node-only pode bater em constraints de edge.

## Como funciona

| Framework | Modelo | Use case típico | Maturidade | Trade-offs |
| --- | --- | --- | --- | --- |
| Express | Middleware-based, minimalista | Microsserviços simples, prototipagem, glue code | Maduro, v5.x | Pouca estrutura; precisa montar validation, DI e contracts manualmente |
| NestJS | Opinativo, DI, decorators | Apps enterprise, time grande, domínio complexo | Maduro, v10+ | Curva de aprendizado; overhead em apps simples |
| Fastify | Schema-first, performance-focused | APIs com contrato claro e throughput alto | Maduro, v5.x | Plugin encapsulation exige mental model; ecossistema menor que Express |
| Hono | Ultralight, edge-first, Fetch API | Edge workers, serverless, multi-runtime | Mais recente, v4+ | Ecossistema menor; edge limita APIs Node-specific |

```typescript
// Express 5
import express from "express";

const app = express();
app.get("/hello", (_req, res) => res.json({ greeting: "hello" }));
app.listen(3000);
```

```typescript
// NestJS
@Controller()
export class AppController {
  @Get("/hello")
  hello() {
    return { greeting: "hello" };
  }
}
```

```typescript
// Fastify
import Fastify from "fastify";

const app = Fastify();
app.get("/hello", async () => ({ greeting: "hello" }));
await app.listen({ port: 3000 });
```

```typescript
// Hono
import { Hono } from "hono";

const app = new Hono();
app.get("/hello", (c) => c.json({ greeting: "hello" }));
export default app;
```

## Na prática

- Microsserviço I/O-bound simples, time pequeno: Express ou Fastify.
- App enterprise, time grande, DI complexa: NestJS.
- API com schema bem definido e throughput alto: Fastify.
- Edge worker, serverless ou multi-runtime: Hono.
- Comparação detalhada: [[12 - Decision tree + cheatsheet]].

## Armadilhas

1. Escolher por hype, não por fit: "todo mundo usa NestJS" não justifica projeto pequeno.
2. Migrar framework no meio do projeto sem motivo forte: custo alto e risco de regressão.
3. Comparar performance por benchmark sintético e ignorar DB, rede, payload e observability.
4. Achar que NestJS é só "Express com decorators": o modelo de módulos e DI muda a arquitetura.

## Em entrevista

"Node has four main framework models in 2026. Express is middleware-based and ubiquitous; NestJS is opinionated, decorator-based, and built around dependency injection; Fastify is schema-first and performance-focused; Hono is ultralight, multi-runtime, and edge-first. The decision is matching, not ranking: pick based on deploy target, domain complexity, team size, and API contract needs."

Vocabulário-chave:

- middleware-based -> baseado em middleware
- opinionated -> opinativo
- schema-first -> orientado por schema
- edge-first -> pensado para edge
- decision matching -> matching entre problema e ferramenta

## Fontes

- [Express](https://expressjs.com/)
- [NestJS docs](https://docs.nestjs.com/)
- [Fastify](https://fastify.dev/)
- [Hono docs](https://hono.dev/docs)

## Veja também

- [[Frameworks e arquitetura]]
- [[02 - Express idiomático]]
- [[03 - NestJS - fundamentos]]
- [[05 - Fastify - schema-first, plugins, performance]]
- [[06 - Hono e edge runtimes]]
- [[12 - Decision tree + cheatsheet]]
- [[Node.js]]

