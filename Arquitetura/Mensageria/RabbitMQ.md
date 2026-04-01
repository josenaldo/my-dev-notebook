---
title: "RabbitMQ"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - arquitetura
  - mensageria
  - entrevista
publish: false
---

# RabbitMQ

Message broker tradicional — routing flexível, múltiplos protocolos, e simplicidade operacional.

## O que é

RabbitMQ é um message broker open-source que implementa AMQP (Advanced Message Queuing Protocol). Diferente do Kafka (event log), RabbitMQ é uma fila tradicional onde mensagens são consumidas e removidas. Destaca-se pelo routing flexível via exchanges.

## Como funciona

### Arquitetura

```text
Producer → Exchange → Binding → Queue → Consumer
```

- **Exchange:** recebe mensagens e roteia para filas baseado em regras
- **Queue:** armazena mensagens até serem consumidas
- **Binding:** regra que conecta exchange a queue
- **Consumer:** processa mensagens da fila

### Tipos de Exchange

| Exchange | Routing | Use case |
| --- | --- | --- |
| Direct | Exact match na routing key | Roteamento simples por chave |
| Fanout | Todas as filas ligadas | Broadcast / pub-sub |
| Topic | Pattern match (`order.*`, `#.error`) | Routing por padrão |
| Headers | Match por headers da mensagem | Routing complexo sem key |

### Garantias

- **Acks:** consumidor confirma processamento. Sem ack = mensagem retorna à fila.
- **Persistent messages:** gravadas em disco (sobrevivem restart do broker)
- **Publisher confirms:** produtor recebe confirmação que o broker recebeu a mensagem
- **Dead Letter Exchange (DLX):** mensagens rejeitadas ou expiradas são roteadas para outra fila

## Quando usar

- **Task queues:** background jobs, email, processamento assíncrono
- **Routing complexo:** quando precisa rotear mensagens por padrões
- **RPC assíncrono:** request-reply via filas
- **Prioridade:** filas com mensagens de diferentes prioridades
- **Simplicidade:** mais fácil de operar que Kafka para cenários simples

## Quando NÃO usar

- **Event replay:** mensagens são removidas após consumo
- **Alto throughput:** Kafka escala melhor para milhões de msg/s
- **Múltiplos consumidores independentes:** cada mensagem vai para um consumidor (ou precisa de fanout explícito)

## Comparação com Kafka

| Aspecto | RabbitMQ | Kafka |
| --- | --- | --- |
| Modelo | Message queue (consume & delete) | Event log (append & retain) |
| Routing | Flexível (exchanges) | Simples (topic + partition) |
| Retenção | Até consumo | Configurável (horas-infinito) |
| Replay | Não | Sim |
| Throughput | Bom (~100K msg/s) | Muito alto (~1M+ msg/s) |
| Ordenação | Por fila | Por partição |
| Operação | Mais simples | Mais complexo (ZooKeeper/KRaft) |

## How to explain in English

"RabbitMQ is a traditional message broker that I use for task processing and routing scenarios. Its exchange-based routing is more flexible than Kafka's topic-partition model — I can route messages based on patterns, headers, or exact keys.

I choose RabbitMQ over Kafka when I need simple task queues, priority-based processing, or complex routing rules, and when I don't need event replay or very high throughput."

### Key vocabulary

- exchange → exchange: componente que roteia mensagens
- fila → queue: armazena mensagens
- binding → binding: regra de roteamento
- acknowledgment → ack: confirmação de processamento
- fila de mensagens mortas → dead letter queue (DLQ/DLX)

## Recursos

- [RabbitMQ Docs](https://www.rabbitmq.com/docs) — documentação oficial
- [RabbitMQ Tutorials](https://www.rabbitmq.com/tutorials) — guias práticos

## Veja também

- [[Mensageria]]
- [[Kafka]]
- [[BullMQ]]
