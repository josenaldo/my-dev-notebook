---
title: "Node.js"
created: 2026-04-01
updated: 2026-04-11
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

```text
┌──────────────────────────────────┐
│    Seu código JavaScript         │
├──────────────────────────────────┤
│    Node.js APIs (fs, http, ...)  │
├──────────────────────────────────┤
│    Node Bindings (C++)           │
├──────────────────────────────────┤
│    V8 Engine   │   libuv          │
│    (JS → ASM)  │   (event loop,  │
│                │    I/O, threads) │
├──────────────────────────────────┤
│    OS (Linux, macOS, Windows)    │
└──────────────────────────────────┘
```

- **V8** — engine JavaScript do Chrome. Compila JS para código nativo via JIT.
- **libuv** — biblioteca C que provê event loop, thread pool, file I/O, networking, timers cross-platform.
- **Node bindings** — pontes C++ entre JS e APIs do OS.
- **Single-threaded event loop** — seu código JS roda em uma thread. I/O é delegado.
- **Thread pool (libuv)** — 4 threads default (configurável via `UV_THREADPOOL_SIZE`). Usada para filesystem, DNS, crypto, compression.

### Single-threaded com non-blocking I/O

**Chave do modelo Node:** I/O **não bloqueia** a thread JS. O Node delega ao OS (via libuv) e registra um callback. Quando o I/O completa, o callback é enfileirado no event loop.

```javascript
// Thread 1 (main) executa isso
fs.readFile('big.txt', (err, data) => {
    console.log('done');  // callback quando I/O completar
});
console.log('continuando');  // imprime ANTES de "done"
```

**Consequência:** uma única thread pode gerenciar milhares de conexões simultâneas — o que seria custoso com um modelo thread-per-request tradicional.

### Event loop phases — detalhado

Ver também [[JavaScript Fundamentals]] (seção event loop). Resumo das fases do libuv:

```
   ┌───────────────────────────┐
┌─►│           timers          │  setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  I/O errors retidos
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  interno
│  └─────────────┬─────────────┘    ┌───────────────┐
│  ┌─────────────┴─────────────┐    │   incoming:   │
│  │           poll            │◄───┤ connections,  │
│  └─────────────┬─────────────┘    │   data, etc.  │
│  ┌─────────────┴─────────────┐    └───────────────┘
│  │           check           │  setImmediate
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  'close' events
   └───────────────────────────┘
```

**Entre cada fase** — microtasks: `process.nextTick` (prioridade máxima), depois Promise callbacks.

**`setImmediate` vs `setTimeout(fn, 0)`:**

```javascript
// Em contexto I/O, setImmediate roda primeiro
fs.readFile('file', () => {
    setTimeout(() => console.log('timeout'), 0);
    setImmediate(() => console.log('immediate'));
    // → 'immediate' primeiro (próxima fase após poll)
});

// Fora de I/O, ordem é imprevisível
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
```

**`process.nextTick` vs `queueMicrotask` vs `setImmediate`:**

| Função | Quando roda |
| --- | --- |
| `process.nextTick` | **Antes** do próximo event loop iteration. Prioridade máxima. |
| `queueMicrotask` | Após código síncrono, entre fases. Como Promise.then. |
| `setImmediate` | Na fase `check` do próximo iteration. |
| `setTimeout(fn, 0)` | Na fase `timers` quando o tempo passar (>= 1ms). |

**Cuidado com `process.nextTick`:** recursão em `nextTick` pode **bloquear o event loop** (nunca avança para próxima fase).

### Worker Threads, cluster, child_process — as 3 formas de paralelismo

| | Worker Threads | Cluster | child_process |
| --- | --- | --- | --- |
| **Uso** | CPU-bound tasks | Escalar HTTP server por CPU | Executar processo externo |
| **Memória** | Separada, mensagens via postMessage | Separada, IPC | Separada, stdio |
| **Criação** | Rápida (~ms) | Lenta (novo processo) | Lenta (novo processo) |
| **Compartilhamento** | SharedArrayBuffer | Via IPC | Via stdio/IPC |
| **Use case** | Image processing, crypto, ML | Servidor web escalado | Rodar comando shell |

**Worker Threads — para CPU-bound:**

```typescript
// worker.ts
import { parentPort, workerData } from 'node:worker_threads';

const result = heavyComputation(workerData);
parentPort?.postMessage(result);

// main.ts
import { Worker } from 'node:worker_threads';

function runWorker(data: any) {
    return new Promise((resolve, reject) => {
        const worker = new Worker('./worker.js', { workerData: data });
        worker.on('message', resolve);
        worker.on('error', reject);
        worker.on('exit', code => {
            if (code !== 0) reject(new Error(`Worker stopped with code ${code}`));
        });
    });
}

const result = await runWorker({ input: 'heavy' });
```

**Cluster — para escalar HTTP:**

```typescript
import cluster from 'node:cluster';
import os from 'node:os';

if (cluster.isPrimary) {
    const numCPUs = os.cpus().length;
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    cluster.on('exit', (worker) => {
        console.log(`Worker ${worker.process.pid} died, restarting`);
        cluster.fork();
    });
} else {
    // Worker process
    startServer();
}
```

**Em produção moderna:** cluster nativo é menos usado. Orchestradores (Kubernetes, PM2) fazem melhor: restart automático, rolling updates, health checks.

**`child_process` — executar comandos:**

```typescript
import { exec, spawn, fork } from 'node:child_process';
import { promisify } from 'node:util';

// exec — buffer completo do output
const execP = promisify(exec);
const { stdout } = await execP('git log --oneline -5');

// spawn — streaming
const proc = spawn('ffmpeg', ['-i', 'input.mp4', 'output.webm']);
proc.stdout.on('data', chunk => process.stdout.write(chunk));
proc.on('close', code => console.log(`exited ${code}`));

// fork — child Node.js com IPC
const child = fork('./worker.js');
child.send({ type: 'work', data: ... });
child.on('message', msg => console.log(msg));
```

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

### Streams — deep dive

Streams são a **abstração fundamental** de Node.js para processar dados em chunks, sem carregar tudo em memória. Essencial para arquivos grandes, network, stdout/stdin.

**4 tipos de streams:**

| Tipo | Uso | Exemplo |
| --- | --- | --- |
| **Readable** | Fonte de dados | `fs.createReadStream`, `process.stdin`, HTTP response |
| **Writable** | Destino de dados | `fs.createWriteStream`, `process.stdout`, HTTP request |
| **Duplex** | Ambos (separados) | TCP socket |
| **Transform** | Duplex que transforma | zlib, crypto, parsers |

**API moderna — `stream/promises` + `pipeline`:**

```typescript
import { createReadStream, createWriteStream } from 'node:fs';
import { pipeline } from 'node:stream/promises';
import { Transform } from 'node:stream';

// Pipeline correto — fecha tudo, propaga erros
await pipeline(
    createReadStream('input.csv'),
    new Transform({
        transform(chunk, encoding, callback) {
            const processed = processChunk(chunk.toString());
            callback(null, processed);
        }
    }),
    createWriteStream('output.csv')
);
// Se qualquer stream errar, pipeline propaga e fecha tudo
```

**Por que `pipeline` e não `.pipe()`:**

```typescript
// RUIM — .pipe() não propaga erros
source.pipe(transform).pipe(destination);
// se transform falha, destination fica aberto, vazamento

// BOM — pipeline lida com cleanup e errors
await pipeline(source, transform, destination);
```

### Backpressure

Quando o consumer é mais lento que o producer, o buffer enche. Streams do Node gerenciam isso automaticamente se você usa `pipe`/`pipeline` ou respeita `drain`:

```typescript
// Manual — respeitar drain
function writeMany(writable: NodeJS.WritableStream) {
    let i = 1_000_000;
    function write() {
        let ok = true;
        while (i > 0 && ok) {
            ok = writable.write(`${i}\n`);
            i--;
        }
        if (i > 0) {
            writable.once('drain', write);  // retoma quando buffer esvaziar
        } else {
            writable.end();
        }
    }
    write();
}
```

### Web Streams vs Node streams

Node suporta **Web Streams** (padrão universal) desde v18:

```typescript
import { Readable } from 'node:stream';

// Web Stream → Node Stream
const webStream = new ReadableStream({ start(controller) { /* ... */ } });
const nodeStream = Readable.fromWeb(webStream);

// Node Stream → Web Stream
const web = Readable.toWeb(nodeReadable);

// Fetch retorna Web Stream
const response = await fetch('https://example.com/big.json');
const reader = response.body!.getReader();
while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    process(value);
}
```

**Em 2026:** Web Streams são mais portáveis (funcionam em browser, Deno, Bun, Cloudflare Workers). Use quando possível.

### Stream patterns

**Transform stream — line parser:**

```typescript
import { Transform } from 'node:stream';

class LineParser extends Transform {
    private buffer = '';

    _transform(chunk: Buffer, encoding: string, callback: Function) {
        this.buffer += chunk.toString();
        const lines = this.buffer.split('\n');
        this.buffer = lines.pop() || '';  // última linha pode estar incompleta
        for (const line of lines) {
            this.push(line);
        }
        callback();
    }

    _flush(callback: Function) {
        if (this.buffer) this.push(this.buffer);
        callback();
    }
}

await pipeline(
    createReadStream('big.csv'),
    new LineParser(),
    new Transform({
        objectMode: true,
        transform(line, enc, cb) {
            const parsed = parseCSV(line);
            cb(null, JSON.stringify(parsed) + '\n');
        }
    }),
    createWriteStream('out.jsonl')
);
```

**Async iteration de streams:**

```typescript
// Modern — for await of
async function processLines(path: string) {
    const stream = createReadStream(path, { encoding: 'utf8' });
    let buffer = '';
    for await (const chunk of stream) {
        buffer += chunk;
        const lines = buffer.split('\n');
        buffer = lines.pop() || '';
        for (const line of lines) {
            await processLine(line);  // processa sequencialmente
        }
    }
}
```

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
- [[Senda JS-TS]] — trilha de aprendizado

## Veja também

- [[JavaScript Fundamentals]] — linguagem, event loop, async
- [[TypeScript]] — tipagem em Node
- [[Testes em JavaScript]] — Vitest, MSW, built-in test runner
- [[React]] — frontend companion
- [[API Design]] — REST, JWT, contratos
- [[Full Stack Open - Guia de Revisão]] — partes 3, 4 sobre Node + Express
- [[System Design]] — troubleshooting cross-stack, building blocks
- [[Kafka]] — Node.js com Kafka
- [[BullMQ]] — job queue em Node.js
- [[Arquitetura de Software]] — Clean Architecture em Node
