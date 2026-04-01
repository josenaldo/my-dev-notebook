---
title: "Mensageria"
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

# Mensageria

Comunicação assíncrona entre sistemas via filas e tópicos de mensagens.

## O que é

Mensageria é o padrão de comunicação onde produtores enviam mensagens para um broker intermediário, e consumidores as processam de forma assíncrona. Desacopla sistemas, melhora resiliência, e permite processamento em escala.

## Como funciona

### Message Queue vs Event Streaming

| Aspecto | Message Queue | Event Streaming |
| --- | --- | --- |
| Modelo | Mensagem consumida e removida | Evento persistido e replayable |
| Exemplo | RabbitMQ, SQS, BullMQ | Kafka, Redpanda, Pulsar |
| Semântica | Task/job processing | Event log, event sourcing |
| Retenção | Até consumo | Configurável (horas, dias, infinito) |
| Consumidores | Um por mensagem (competing) | Múltiplos (cada um com seu offset) |
| Use case | Background jobs, email, notificações | Event-driven architecture, analytics, CQRS |

### Padrões de mensageria

**Point-to-Point:** uma mensagem vai para um consumidor. Típico de filas.

**Pub/Sub:** uma mensagem vai para todos os subscribers. Típico de tópicos.

**Fan-out:** uma mensagem dispara múltiplas ações em paralelo.

**Dead Letter Queue (DLQ):** fila para mensagens que falharam após N retries.

### Garantias de entrega

| Garantia | Descrição | Trade-off |
| --- | --- | --- |
| At-most-once | Pode perder, nunca duplica | Performance máxima |
| At-least-once | Nunca perde, pode duplicar | Consumidor deve ser idempotente |
| Exactly-once | Nem perde, nem duplica | Complexo, mais lento (transações) |

**Na prática:** at-least-once + idempotência no consumidor é o padrão mais comum.

## Quando usar

- **Desacoplamento:** serviços não precisam se conhecer diretamente
- **Resiliência:** se o consumidor cai, mensagens ficam na fila
- **Escala:** adicionar mais consumidores para processar mais
- **Background jobs:** email, relatórios, processamento de imagem
- **Event-driven:** reagir a eventos de domínio (pedido criado → notificar → atualizar estoque)

## Armadilhas comuns

- **Mensagens duplicadas:** at-least-once requer idempotência. Usar chave de idempotência.
- **Ordem de processamento:** filas gerais não garantem ordem. Kafka garante por partição.
- **Poison messages:** mensagem que sempre falha. Implementar DLQ e alertas.
- **Over-engineering:** nem tudo precisa de fila. Chamada HTTP síncrona é mais simples quando funciona.

## How to explain in English

"Message queues and event streaming are the backbone of distributed systems. I choose between them based on the communication pattern: queues for task processing where each message is handled once, and event streaming when multiple consumers need to react to the same event or when I need event replay capability.

The most important principle is designing for at-least-once delivery with idempotent consumers. This gives us reliability without the complexity of exactly-once semantics."

### Key vocabulary

- fila de mensagens → message queue
- fluxo de eventos → event streaming
- produtor → producer / publisher
- consumidor → consumer / subscriber
- broker → message broker
- fila de mensagens mortas → dead letter queue (DLQ)
- idempotência → idempotency

## Veja também

- [[Event Streaming]]
- [[Kafka]]
- [[RabbitMQ]]
- [[BullMQ]]
- [[System Design]]
