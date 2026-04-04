---
title: "Node.js"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - javascript
  - backend
  - entrevista
publish: false
---

# Node.js

Runtime JavaScript para backend — event-driven, non-blocking I/O, e o ecossistema npm.

## O que é

Node.js é um runtime JavaScript baseado no V8 engine do Chrome, projetado para I/O assíncrono e non-blocking. Ideal para APIs, real-time apps, e microserviços I/O-bound. Para entrevistas, o foco é em: event loop, streams, error handling, e quando Node é (ou não) a escolha certa.

## Como funciona

### Arquitetura

```text
JavaScript Code
    ↓
V8 Engine (compila JS → código nativo)
    ↓
Node.js Bindings (C++)
    ↓
libuv (event loop, thread pool para I/O)
    ↓
OS (network, filesystem, etc.)
```

- **Single-threaded:** o event loop roda em uma thread. I/O é delegado ao OS ou thread pool do libuv.
- **Worker Threads:** para CPU-bound tasks (parsing, criptografia). Não para I/O.
- **Cluster:** múltiplos processos Node compartilhando a mesma porta. Usa todos os cores.

### Frameworks

| Framework | Estilo | Use case |
| --- | --- | --- |
| Express | Minimalista, middleware-based | APIs simples, prototipagem |
| NestJS | Opinativo, inspirado em Angular/Spring | Apps enterprise, microserviços |
| Fastify | Performance-first, schema-based | APIs de alta performance |
| Hono | Ultralight, edge-first | Serverless, edge computing |

**NestJS** é o mais próximo do Spring Boot em filosofia: DI, decorators, módulos, guards, interceptors.

### Error Handling

```typescript
// Express middleware de erro
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  logger.error(err.stack);
  res.status(500).json({
    type: "https://api.example.com/errors/internal",
    title: "Internal Server Error",
    status: 500,
    detail: err.message
  });
});

// Async error handling (Express não captura automaticamente)
const asyncHandler = (fn: Function) =>
  (req: Request, res: Response, next: NextFunction) =>
    Promise.resolve(fn(req, res, next)).catch(next);

app.get("/users/:id", asyncHandler(async (req, res) => {
  const user = await userService.findById(req.params.id);
  if (!user) throw new NotFoundError("User not found");
  res.json(user);
}));
```

### Streams

Processar dados em chunks sem carregar tudo em memória.

```typescript
import { createReadStream, createWriteStream } from "fs";
import { pipeline } from "stream/promises";
import { Transform } from "stream";

// Processar arquivo grande linha por linha
await pipeline(
  createReadStream("input.csv"),
  new Transform({
    transform(chunk, encoding, callback) {
      const processed = processChunk(chunk.toString());
      callback(null, processed);
    }
  }),
  createWriteStream("output.csv")
);
```

### npm e Package Management

- **package.json:** dependências, scripts, metadata
- **package-lock.json:** lock de versões exatas (comitar no git!)
- **Semantic versioning:** `^1.2.3` = compatível com 1.x.x, `~1.2.3` = compatível com 1.2.x
- **npm vs yarn vs pnpm:** pnpm é mais eficiente (hard links), npm é o padrão

## Quando usar

- **Node.js:** APIs REST/GraphQL, real-time (WebSocket), microserviços I/O-bound, BFF (Backend for Frontend)
- **Não usar:** CPU-bound heavy (image processing, ML inference) — melhor com Python ou Go
- **NestJS:** quando precisa de estrutura enterprise, DI, e padrões familiares do Spring
- **Express:** APIs simples, prototipagem rápida

## Armadilhas comuns

- **Blocking the event loop:** operações CPU-intensive na thread principal (crypto sync, JSON.parse de payloads enormes). Usar Worker Threads.
- **Unhandled promise rejections:** promises sem `.catch()` ou try/catch em async. Configurar `process.on('unhandledRejection')`.
- **Callback hell:** aninhar callbacks. Usar async/await.
- **Memory leaks:** event listeners não removidos, closures que retêm referências grandes, caches sem TTL.
- **`require` vs `import`:** CJS (`require`) é síncrono, ESM (`import`) é assíncrono. Projetos novos devem usar ESM.

## Na prática (da minha experiência)

> Em projetos fullstack, uso Node.js com NestJS ou Express como BFF (Backend for Frontend) que agrega dados de microserviços Java. A vantagem é compartilhar types TypeScript entre frontend React e backend Node. Para APIs que fazem muitas chamadas a serviços externos (I/O-bound), Node.js performa melhor que Java com menos recursos, graças ao event loop non-blocking.

## How to explain in English

"Node.js is my choice for I/O-bound backend services. Its event loop model means a single process can handle thousands of concurrent connections without the overhead of thread management. This is particularly effective for API gateways or BFF services that aggregate data from multiple microservices.

I typically use NestJS for larger applications because it provides structure similar to Spring Boot — dependency injection, modules, guards, and interceptors. For simpler services, Express with TypeScript is my go-to.

One critical thing about Node.js is knowing when NOT to use it. CPU-intensive operations block the event loop and degrade performance for all concurrent requests. For heavy computation, I'd use Worker Threads or offload to a different service. In a microservices architecture, I use Node.js for the API layer and Java for services that need heavy processing or complex business logic.

Error handling in Node.js requires attention — especially with async code. I always use a global error handler middleware and wrap async route handlers to catch unhandled promise rejections. In NestJS, exception filters handle this elegantly."

### Key vocabulary

- laço de eventos → event loop
- I/O não-bloqueante → non-blocking I/O
- thread de trabalho → worker thread
- fluxo de dados → stream
- middleware → middleware: função que intercepta requisições
- gerenciador de pacotes → package manager (npm, pnpm)
- módulo → module: unidade de código importável

### ORMs

**Sequelize** — ORM para Node.js com suporte a PostgreSQL, MySQL, SQLite:

```typescript
// Model
@Table
class Patient extends Model {
  @Column declare name: string;
  @Column declare email: string;
  @HasMany(() => Appointment) declare appointments: Appointment[];
}

// Query
const patients = await Patient.findAll({
  where: { active: true },
  include: [Appointment],  // eager loading (evita N+1)
  limit: 20,
  offset: 0,
});
```

**Cuidados com Sequelize:**
- Associações mal configuradas causam queries inesperadas
- Paginação: usar `limit` + `offset` ou cursor-based para datasets grandes
- Migrations versionadas (similar ao Flyway do Java)

**Alternativas:** Prisma (type-safe, schema-first), TypeORM (decorators, similar ao JPA), Drizzle (lightweight).

> **Fontes:**
> - [Sequelize](https://sequelize.org/)
> - [Sequelize Best Practices](https://climbtheladder.com/10-sequelize-js-best-practices/)
> - [Mastering Pagination in Sequelize](https://medium.com/hexaworks-papers/mastering-pagination-in-sequelize-e50dbc9de01d)

## Recursos

- [Node.js Docs](https://nodejs.org/docs/latest/api/) — documentação oficial
- [NestJS Docs](https://docs.nestjs.com/) — framework enterprise
- [Full Stack Open](https://fullstackopen.com/en/) — curso gratuito de fullstack com Node/React
- [Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript) — boas práticas
- [Mastering Node.js and Express](https://dev.to/hanzla-mirza/mastering-nodejs-and-expressjs-advanced-tips-tricks-and-high-level-insights-183j)
- [[Trilha JS-TS]] — trilha de aprendizado

## Veja também

- [[JavaScript Fundamentals]]
- [[TypeScript]]
- [[React]]
- [[API Design]]
