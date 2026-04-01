---
title: "BullMQ"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - arquitetura
  - mensageria
  - javascript
  - entrevista
publish: false
---

# BullMQ

Biblioteca de filas para Node.js baseada em Redis — background jobs, scheduling, e rate limiting.

## O que é

BullMQ é uma biblioteca Node.js/TypeScript para processamento de jobs em filas, usando Redis como backend. É a evolução do Bull, com suporte a flows, rate limiting, e melhor performance. Ideal para background processing em aplicações Node.js sem precisar de um broker externo pesado como Kafka ou RabbitMQ.

## Como funciona

### Conceitos básicos

```typescript
import { Queue, Worker } from 'bullmq';
import { Redis } from 'ioredis';

const connection = new Redis();

// Criar fila
const emailQueue = new Queue('emails', { connection });

// Adicionar job
await emailQueue.add('send-welcome', {
  to: 'paciente@email.com',
  template: 'welcome',
  data: { name: 'Josenaldo' }
});

// Processar jobs
const worker = new Worker('emails', async (job) => {
  await sendEmail(job.data.to, job.data.template, job.data.data);
}, { connection });

// Eventos
worker.on('completed', (job) => console.log(`Job ${job.id} completed`));
worker.on('failed', (job, err) => console.error(`Job ${job.id} failed`, err));
```

### Features

- **Delayed jobs:** `emailQueue.add('reminder', data, { delay: 3600000 })` — executa em 1h
- **Repeatable jobs:** `{ repeat: { every: 60000 } }` — a cada 1 min (cron jobs)
- **Rate limiting:** `{ limiter: { max: 10, duration: 1000 } }` — max 10 jobs/segundo
- **Retries:** `{ attempts: 3, backoff: { type: 'exponential', delay: 1000 } }`
- **Priorities:** `{ priority: 1 }` — menor número = maior prioridade
- **Flows:** encadear jobs com dependências (parent-child)
- **Concurrency:** `new Worker('queue', handler, { concurrency: 5 })`

### Dashboard

```typescript
// Bull Board para monitoramento
import { createBullBoard } from '@bull-board/api';
import { BullMQAdapter } from '@bull-board/api/bullMQAdapter';

createBullBoard({ queues: [new BullMQAdapter(emailQueue)] });
```

## Quando usar

- **Background jobs em Node.js:** emails, relatórios, processamento de imagem
- **Scheduled tasks:** cron jobs, lembretes, cleanup
- **Rate limiting:** limitar chamadas a APIs externas
- **Filas com prioridade:** processar jobs urgentes primeiro
- **Já usa Redis:** BullMQ não adiciona infraestrutura nova

## Quando NÃO usar

- **Event streaming:** não é log persistente, é fila de jobs. Usar Kafka.
- **Multi-language:** BullMQ é Node.js only. Para Java, usar Kafka ou RabbitMQ.
- **Muito alta escala:** Redis como broker tem limites. Kafka escala melhor.

## Na prática (da minha experiência)

> No MedEspecialista, uso BullMQ para processamento assíncrono no backend Node.js — envio de emails, geração de relatórios, e sincronização com serviços externos. A combinação com Redis (que já usamos para cache) evita adicionar um broker externo. O Bull Board fornece um dashboard para monitorar filas em produção.

## How to explain in English

"BullMQ is my go-to for background job processing in Node.js applications. It uses Redis as a backend, which means no additional infrastructure if you're already using Redis for caching. It supports delayed jobs, scheduling, rate limiting, retries with exponential backoff, and priority queues — all the features you'd expect from a production job queue.

I choose BullMQ when the application is Node.js-only and doesn't need the complexity of Kafka or RabbitMQ. For cross-platform microservices, I'd use Kafka or RabbitMQ instead."

### Key vocabulary

- fila de jobs → job queue
- trabalho em segundo plano → background job
- job agendado → scheduled/delayed job
- limitação de taxa → rate limiting
- tentativas → retries / attempts
- backoff exponencial → exponential backoff

## Recursos

- [BullMQ Docs](https://docs.bullmq.io/) — documentação oficial
- [Bull Board](https://github.com/felixmosh/bull-board) — dashboard de monitoramento

## Veja também

- [[Mensageria]]
- [[Kafka]]
- [[RabbitMQ]]
- [[Node.js]]
