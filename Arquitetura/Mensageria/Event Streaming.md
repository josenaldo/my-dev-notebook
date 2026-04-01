---
title: "Event Streaming"
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

# Event Streaming

Processamento contínuo de fluxos de eventos — a base de arquiteturas event-driven modernas.

## O que é

Event Streaming é o padrão onde eventos (fatos que aconteceram) são publicados em um log persistente e consumidos de forma contínua por um ou mais serviços. Diferente de message queues tradicionais, eventos são retidos e podem ser re-processados.

## Como funciona

### Event-Driven Architecture

```text
Serviço A (produtor)
  → "PedidoCriado" → Event Broker (Kafka)
                         ├── Serviço B (notificação)
                         ├── Serviço C (estoque)
                         └── Serviço D (analytics)
```

Cada serviço reage ao evento independentemente. Se o serviço de estoque cai, quando volta, processa os eventos pendentes.

### Conceitos-chave

- **Event:** fato imutável que aconteceu. `{ type: "OrderCreated", data: { orderId: "123", ... }, timestamp: "..." }`
- **Topic/Stream:** canal onde eventos são publicados, organizado por domínio
- **Partition:** subdivisão de um topic para paralelismo. Ordem garantida dentro da partição.
- **Consumer Group:** grupo de consumidores que divide as partições entre si
- **Offset:** posição do consumidor no log. Permite re-processamento.

### Padrões

**Event Notification:** "algo aconteceu, reaja se quiser". Baixo acoplamento.

**Event-Carried State Transfer:** o evento carrega todos os dados necessários. Consumidor não precisa consultar o produtor.

**Event Sourcing:** estado reconstruído a partir da sequência de eventos. O log É a fonte de verdade.

**CQRS (Command Query Responsibility Segregation):** modelos separados para escrita (commands) e leitura (queries). Frequentemente combinado com Event Sourcing.

## Quando usar

- **Microserviços desacoplados:** serviços reagem a eventos sem dependência direta
- **Audit trail:** log imutável de tudo que aconteceu
- **Analytics em tempo real:** processar eventos conforme chegam
- **CQRS:** quando leitura e escrita têm necessidades muito diferentes
- **Replay:** re-processar eventos para corrigir bugs ou migrar dados

## Armadilhas comuns

- **Schema evolution:** mudar o formato do evento sem quebrar consumidores. Usar schema registry.
- **Ordenação entre partições:** Kafka garante ordem por partição, não por topic. Escolher partition key com cuidado.
- **Event granularity:** eventos muito genéricos perdem utilidade, muito específicos geram acoplamento

## How to explain in English

"Event streaming is the foundation of the event-driven architectures I've built. The key insight is treating events as immutable facts in a persistent log, rather than transient messages. This enables multiple consumers to process the same events independently, and allows replay for debugging or data migration.

At Muvz, I introduced Kafka for event streaming between microservices. The email notification service, for example, consumes OrderCreated events. If the email service is down, the events stay in Kafka and are processed when it recovers — no messages lost, no complex retry logic in the producer."

### Key vocabulary

- fluxo de eventos → event streaming
- evento → event: fato imutável que aconteceu
- log de eventos → event log
- sourcing de eventos → event sourcing
- partição → partition

## Veja também

- [[Mensageria]]
- [[Kafka]]
- [[System Design]]
- [[Event Storming]]
