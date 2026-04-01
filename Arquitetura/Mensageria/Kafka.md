---
title: "Kafka"
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

# Kafka

Plataforma de event streaming distribuída — alto throughput, persistência, e ordenação por partição.

## O que é

Apache Kafka é uma plataforma de event streaming distribuída criada pelo LinkedIn (2011). Projetada para alto throughput (milhões de mensagens/segundo), durabilidade (mensagens persistidas em disco), e escalabilidade horizontal (partições distribuídas em brokers).

## Como funciona

### Arquitetura

```text
Producers → Kafka Cluster → Consumers
                 │
          ┌──────┼──────┐
          │      │      │
        Broker  Broker  Broker
          │      │      │
        Topics com Partitions
```

- **Broker:** servidor Kafka. Um cluster tem múltiplos brokers.
- **Topic:** canal de mensagens, dividido em partições.
- **Partition:** unidade de paralelismo. Mensagens ordenadas dentro da partição.
- **Replication:** cada partição tem réplicas em diferentes brokers (fault tolerance).
- **Consumer Group:** consumidores que dividem partições entre si.
- **Offset:** posição do consumidor no log.

### Garantias

- **Ordenação:** garantida dentro de uma partição (por key)
- **Durabilidade:** mensagens persistidas em disco com replicação
- **Retenção:** configurável (horas, dias, infinito)
- **At-least-once:** default. Exactly-once possível com transações.

### Kafka Connect e Kafka Streams

- **Kafka Connect:** integração com sistemas externos (banco, S3, Elasticsearch) via conectores prontos
- **Kafka Streams:** processamento de eventos em tempo real como biblioteca Java (sem cluster separado)

## Quando usar

- **Event streaming:** arquitetura event-driven entre microserviços
- **Alto throughput:** milhões de mensagens/segundo
- **Replay:** necessidade de re-processar eventos
- **Log de auditoria:** registro imutável de eventos
- **Integração de dados:** pipeline entre sistemas (CDC com Debezium + Connect)

## Quando NÃO usar

- **Filas simples de background jobs:** RabbitMQ ou BullMQ são mais simples
- **Poucas mensagens:** overhead de operar um cluster Kafka não compensa
- **Precisa de routing complexo:** RabbitMQ tem exchange routing mais flexível

## Na prática (da minha experiência)

> Na Muvz, liderei a adoção de Kafka para comunicação entre 5 microserviços Spring Boot. Construí proof-of-concepts, configurei o cluster em Kubernetes com DevOps, e implementei integrações event-driven incluindo um microserviço de email com logging e auditoria. A decisão por Kafka (vs RabbitMQ) foi baseada na necessidade de replay de eventos e múltiplos consumidores independentes.

## How to explain in English

"Kafka is my choice for event streaming in microservices architectures. I led its adoption at Muvz, where we used it to decouple five Spring Boot microservices. The key advantages were: ordered processing within partitions, message persistence for replay, and the ability for multiple consumer groups to independently process the same events.

The decision to use Kafka over RabbitMQ was driven by our need for event replay — when we deployed a new analytics service, it could process all historical events from the beginning of the topic."

### Key vocabulary

- broker → broker: servidor do cluster Kafka
- tópico → topic: canal de mensagens
- partição → partition: unidade de paralelismo
- grupo de consumidores → consumer group
- offset → offset: posição no log
- replicação → replication: cópias para fault tolerance

## Recursos

- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- Notas detalhadas: [[Kafka Concepts]] (em Java/Backend/Kafka/)

## Veja também

- [[Mensageria]]
- [[Event Streaming]]
- [[RabbitMQ]]
- [[BullMQ]]
- [[System Design]]
