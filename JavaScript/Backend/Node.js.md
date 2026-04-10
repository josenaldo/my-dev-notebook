---
title: "Node.js"
created: 2026-04-01
updated: 2026-04-10
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

## Troubleshooting em produção

Problemas recorrentes em aplicações Node.js — equivalentes aos problemas de Java/Spring Boot, mas com soluções idiomáticas do ecossistema Node.

### Connection pool exausto

**Sintoma:** requests travam, timeout no banco de dados.

**Com knex/pg-pool:**

```typescript
// knexfile.ts
export default {
  pool: {
    min: 2,
    max: 10,
    acquireTimeoutMillis: 30000,  // timeout para adquirir conexão
    idleTimeoutMillis: 10000,     // liberar idle connections
  },
  // Detectar leaks (dev)
  afterCreate: (conn, done) => {
    console.log('Connection created');
    done(null, conn);
  }
};
```

**Com Prisma:**

```typescript
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  // connection_limit é via URL: ?connection_limit=10&pool_timeout=30
}
```

**Causa comum em Node:** não fechar transações. Diferente de Java (que tem `@Transactional`), em Node você gerencia manualmente:

```typescript
// RUIM — conexão nunca devolvida se der erro
const trx = await knex.transaction();
await trx('patients').insert(data);
// se der throw aqui, trx nunca fecha

// BOM — try/catch com rollback
const trx = await knex.transaction();
try {
  await trx('patients').insert(data);
  await trx.commit();
} catch (err) {
  await trx.rollback();
  throw err;
}
```

### N+1 queries

**Com Sequelize:**

```typescript
// RUIM — N+1
const doctors = await Doctor.findAll();
for (const d of doctors) {
  const appointments = await d.getAppointments();  // 1 query por doctor
}

// BOM — eager loading
const doctors = await Doctor.findAll({
  include: [{ model: Appointment }],  // JOIN
});

// BOM — DataLoader pattern (GraphQL)
const appointmentLoader = new DataLoader(async (doctorIds) => {
  const appointments = await Appointment.findAll({
    where: { doctorId: doctorIds },
  });
  return doctorIds.map(id => appointments.filter(a => a.doctorId === id));
});
```

**Com Prisma:**

```typescript
// BOM — include (eager)
const doctors = await prisma.doctor.findMany({
  include: { appointments: true },
});

// BOM — select (projection)
const doctors = await prisma.doctor.findMany({
  select: { name: true, _count: { select: { appointments: true } } },
});
```

### Event loop blocking

**Sintoma:** latência de todas as requests sobe; o servidor "trava" por instantes.

**Causa:** operação CPU-intensive na thread principal (JSON.parse de payload grande, crypto sync, regex complexa).

**Diagnóstico:**

```typescript
// Detectar event loop lag
import { monitorEventLoopDelay } from 'perf_hooks';

const h = monitorEventLoopDelay({ resolution: 50 });
h.enable();

// Exportar para métricas
setInterval(() => {
  console.log(`Event loop p99: ${h.percentile(99) / 1e6}ms`);
  h.reset();
}, 10000);
```

**Soluções:**
- `Worker Threads` para CPU-bound tasks
- `child_process.fork()` para processos isolados
- Cluster mode (`pm2`, `node cluster`) para usar múltiplos cores
- Streaming para processar dados grandes em chunks

### Memory leak

**Diagnóstico:**

```bash
# Iniciar com inspector
node --inspect app.js

# Chrome DevTools → chrome://inspect → Heap Snapshot
# Comparar 2 snapshots separados por tempo → objetos que só crescem

# Ou via CLI
node --expose-gc -e "global.gc(); console.log(process.memoryUsage())"
```

**Causas comuns:**
- Event listeners não removidos (`emitter.on` sem `emitter.off`)
- Closures que capturam objetos grandes
- Cache em memória sem TTL (`Map` que só cresce)
- Timers não limpos (`setInterval` sem `clearInterval`)

```typescript
// RUIM — leak clássico
const cache = new Map();  // cresce para sempre

// BOM — com TTL
import { LRUCache } from 'lru-cache';
const cache = new LRUCache({ max: 1000, ttl: 1000 * 60 * 5 }); // 5 min
```

### Graceful shutdown

```typescript
// Express
const server = app.listen(3000);

process.on('SIGTERM', () => {
  console.log('SIGTERM received, draining...');
  server.close(() => {  // para de aceitar novas, espera em andamento
    // fechar conexões de banco, Redis, etc.
    prisma.$disconnect();
    process.exit(0);
  });

  // força saída após timeout
  setTimeout(() => process.exit(1), 30000);
});

// NestJS — built-in
app.enableShutdownHooks();  // dispara onModuleDestroy nos providers
```

### Circuit breaker

```typescript
// Com opossum
import CircuitBreaker from 'opossum';

const breaker = new CircuitBreaker(callExternalService, {
  timeout: 3000,           // timeout de 3s
  errorThresholdPercentage: 50,  // abre após 50% de falhas
  resetTimeout: 30000,     // tenta half-open após 30s
});

breaker.fallback(() => ({ cached: true, data: getCachedData() }));

breaker.on('open', () => metrics.increment('circuit.open'));

const result = await breaker.fire(requestData);
```

→ Para comparação cross-stack: [[System Design]] (seção Problemas comuns em produção)

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
- [[System Design]] — troubleshooting cross-stack, building blocks
