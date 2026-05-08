---
title: "Node.js"
created: 2026-04-01
updated: 2026-05-08
type: concept
status: evergreen
tags:
  - javascript
  - backend
  - entrevista
publish: false
---

# Node.js

Deep dive em **Node.js** como runtime JavaScript para backend — event-driven, non-blocking I/O, ecossistema npm. Para fundamentos da linguagem (event loop, closures, async), ver [[JavaScript Fundamentals]]. Para TypeScript, ver [[TypeScript]]. Para testes, ver [[Testes em JavaScript]].

## O que é

Node.js é um runtime JavaScript criado por Ryan Dahl em 2009, baseado no **V8 engine** do Chrome com **libuv** para I/O assíncrono. Projetado para servidores I/O-bound de alta concorrência, é a base da maioria dos backends JavaScript modernos, ferramentas (build tools, CLIs) e package management.

**Em 2026:**

- **Node 22 LTS** é o current stable, Node 24 iteration current
- **Suporte nativo a TypeScript** via `--experimental-strip-types` (22.18+), estável em 24+
- **ESM é default** para novos projetos
- **Bun** (agora da Anthropic) e **Deno** são concorrentes sérios
- **Virtual Threads na JVM** e Go tornaram Node menos dominante em I/O-bound puro, mas ecossistema npm continua imbatível
- **test runner nativo** (`node --test`) ganhou features, competitivo com Vitest
- **watch mode nativo** (`node --watch`) substitui nodemon

Em entrevistas, o que diferencia um senior em Node.js:

1. **Event loop phases (libuv)** — timers, pending, poll, check, close
2. **Streams** — Readable, Writable, Transform, Duplex, backpressure
3. **Error handling** — unhandledRejection, uncaughtException, domain (deprecated)
4. **Cluster vs Worker Threads vs child_process** — quando cada um
5. **Frameworks** — Express, Fastify, NestJS, Hono — trade-offs
6. **Performance** — profiling, --inspect, clinic.js, autocannon
7. **Security** — npm audit, supply chain, secrets, input validation
8. **Moderno** — ESM, TypeScript nativo, built-in test runner, watch mode
9. **Integrações** — Postgres, Redis, Kafka, gRPC, GraphQL

## Como funciona

### Arquitetura

> [!nota] Migrado para galho próprio
> A anatomia interna do runtime — V8, libuv, thread pool — foi expandida em [[Runtime e Event Loop]]. Veja em particular [[02 - V8, libuv e thread pool]] (componentes), [[01 - Single-thread e non-blocking I-O]] (modelo), e [[07 - I-O assíncrono - kernel vs thread pool]] (onde o paralelismo verdadeiro mora).

### Single-threaded com non-blocking I/O

> [!nota] Migrado para galho próprio
> O modelo single-thread + non-blocking I/O foi expandido em [[01 - Single-thread e non-blocking I-O]] dentro do galho [[Runtime e Event Loop]].

### Event loop phases — detalhado

> [!nota] Migrado para galho próprio
> As fases do event loop foram expandidas em [[Runtime e Event Loop]]. Veja em particular [[04 - As fases do event loop]] (ciclo libuv), [[05 - Microtasks - nextTick, queueMicrotask, Promise.then]] (microtasks), e [[06 - Macrotasks e timers - setTimeout, setInterval, setImmediate]] (timers).

### Worker Threads, cluster, child_process — as 3 formas de paralelismo

> [!nota] Migrado para galho próprio
> As 3 ferramentas de paralelismo foram expandidas em [[Paralelismo]] (galho 2). Veja em particular [[02 - As 3 ferramentas - Worker Threads, Cluster, child_process]] (visão geral comparativa), [[03 - Worker Threads - fundamentos]] (Worker Threads), [[07 - Cluster - escalando HTTP por CPU]] (Cluster), [[08 - child_process com exec e spawn]] (rodar comandos externos) e [[09 - child_process com fork - Node child com IPC]] (Node child isolado). Decision tree completa em [[11 - Decision tree - qual ferramenta para qual problema]].

### Frameworks

> [!nota] Migrado para galho próprio
> Os 4 frameworks principais foram expandidos em [[Frameworks e arquitetura]] (galho 4). Veja em particular [[01 - Os 4 frameworks - Express, NestJS, Fastify, Hono]] (visão geral comparativa), [[02 - Express idiomático]] (Express moderno), [[03 - NestJS - fundamentos]] (DI + módulos), [[05 - Fastify - schema-first, plugins, performance]] (Fastify) e [[06 - Hono e edge runtimes]] (edge-first).

### Error Handling

> [!nota] Migrado para galho próprio
> Error handling estruturado (Problem Details RFC 7807) foi expandido em [[08 - Error handling estruturado]] no galho [[Frameworks e arquitetura]], com implementação em todos os 4 frameworks principais.

### Streams — deep dive

> [!nota] Migrado para galho próprio
> Os 4 tipos de stream foram expandidos em [[Streams]] (galho 3). Veja em particular [[02 - Os 4 tipos - Readable, Writable, Duplex, Transform]] (visão geral comparativa), [[03 - Readable streams]] (Readable detalhado), [[04 - Writable streams]] (Writable detalhado) e [[05 - Duplex e Transform]] (Duplex vs Transform).

### Backpressure

> [!nota] Migrado para galho próprio
> Backpressure como mecânica explícita foi expandido em [[06 - Backpressure]] no galho [[Streams]].

### Web Streams vs Node streams

> [!nota] Migrado para galho próprio
> Interop Node ↔ Web Streams foi expandido em [[09 - Web Streams - interop com padrão universal]] no galho [[Streams]].

### Stream patterns

> [!nota] Migrado para galho próprio
> Padrões práticos (line parser, CSV → JSONL, fetch streaming, multipart, tee) foram expandidos em [[10 - Padrões práticos]] no galho [[Streams]]. Decision tree e armadilhas em [[12 - Armadilhas, regras práticas, cheatsheet]].

### npm e Package Management

- **package.json:** dependências, scripts, metadata
- **package-lock.json:** lock de versões exatas (comitar no git!)
- **Semantic versioning:** `^1.2.3` = compatível com 1.x.x, `~1.2.3` = compatível com 1.2.x
- **npm vs yarn vs pnpm vs bun:** em 2026, **pnpm é o mais eficiente** (hard links), **bun** tem instalação mais rápida, **npm** é o padrão default

### Node moderno — features que você deveria usar

**TypeScript nativo (Node 22.18+):**

```bash
node --experimental-strip-types app.ts
# Node 24+ estável
```

Sem precisar de `tsx`, `ts-node`, `swc-node`. Type stripping — remove tipos, roda JS puro.

**Built-in test runner:**

```typescript
// test.ts
import { test, describe } from 'node:test';
import assert from 'node:assert/strict';

describe('math', () => {
    test('add', () => {
        assert.equal(2 + 2, 4);
    });

    test('divide', (t) => {
        t.skip('wip');
    });
});
```

```bash
node --test
node --test --watch
node --test-reporter=spec
```

Competitivo com Vitest para projetos simples. Para Testing Library, ainda use Vitest/Jest. Ver [[Testes em JavaScript]].

**Watch mode nativo:**

```bash
node --watch app.js
# substitui nodemon
```

**Env file loading:**

```bash
node --env-file=.env app.js
# carrega .env automaticamente, sem dotenv
```

**`--import` para pré-carregamento:**

```bash
node --import ./register-hooks.mjs app.mjs
```

**Sea (Single Executable Apps):**

```bash
node --experimental-sea-config sea-config.json
# compila seu app em um único binário standalone
```

**`fs/promises`, `stream/promises`, `timers/promises`:** versões Promise-based dos módulos core.

```typescript
import { readFile } from 'node:fs/promises';
import { setTimeout } from 'node:timers/promises';

const data = await readFile('file.txt', 'utf8');
await setTimeout(1000);  // delay
```

## Quando usar

- **Node.js:** APIs REST/GraphQL, real-time (WebSocket), microserviços I/O-bound, BFF (Backend for Frontend)
- **Não usar:** CPU-bound heavy (image processing, ML inference) — melhor com Python ou Go
- **NestJS:** quando precisa de estrutura enterprise, DI, e padrões familiares do Spring
- **Express:** APIs simples, prototipagem rápida

## Armadilhas comuns

- **Blocking the event loop:** Sintomas, causas e diagnóstico em [[10 - Bloqueio do event loop - sintomas e causas]] e [[11 - Diagnóstico do event loop]] (galho [[Runtime e Event Loop]]).
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

> [!nota] Migrado para galho próprio
> Sintomas, causas e ferramentas de diagnóstico foram expandidos em [[Runtime e Event Loop]]: [[10 - Bloqueio do event loop - sintomas e causas]] e [[11 - Diagnóstico do event loop]].

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
- [[Senda JS-TS]] — trilha de aprendizado

## Veja também

- [[Runtime e Event Loop]] — galho 1 da trilha Node Senior; deep dive do motor (single-thread, libuv, fases, microtasks, async/await, bloqueio, diagnóstico)
- [[Paralelismo]] — galho 2 da trilha Node Senior; as 3 ferramentas de paralelismo (Worker Threads, Cluster, child_process), SharedArrayBuffer/Atomics, pool de workers, decision tree
- [[Streams]] — galho 3 da trilha Node Senior; abstração fundamental para processar dados em chunks (4 tipos, backpressure, pipeline, async iter, Web Streams, padrões práticos)
- [[Frameworks e arquitetura]] — galho 4 da trilha Node Senior; os 4 frameworks principais (Express, NestJS, Fastify, Hono), patterns transversais e arquitetura
- [[JavaScript Fundamentals]] — linguagem, event loop, async
- [[TypeScript]] — tipagem em Node
- [[Testes em JavaScript]] — Vitest, MSW, built-in test runner
- [[React]] — frontend companion
- [[API Design]] — REST, JWT, contratos
- [[Full Stack Open - Guia de Revisão]] — partes 3, 4 sobre Node + Express
- [[System Design]] — troubleshooting cross-stack, building blocks
- [[03-Dominios/Java/Backend/Kafka/Kafka]] — Node.js com Kafka
- [[BullMQ]] — job queue em Node.js
- [[Arquitetura de Software]] — Clean Architecture em Node
