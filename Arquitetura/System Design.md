---
title: "System Design"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - arquitetura
  - entrevista
publish: false
---

# System Design

A habilidade de projetar sistemas escaláveis, confiáveis e manuteníveis — e a skill mais avaliada em entrevistas de senior/staff.

## O que é

System Design é o processo de definir a arquitetura, componentes, interfaces e fluxo de dados de um sistema para atender requisitos funcionais e não-funcionais. Em entrevistas, é uma conversa de 45-60 minutos onde você projeta um sistema do zero, demonstrando capacidade de tomar decisões de trade-off e comunicar claramente.

## Como funciona

### Framework para entrevistas de System Design

**1. Clarificar requisitos (5 min)**

- Requisitos funcionais: o que o sistema faz?
- Requisitos não-funcionais: escala, latência, consistência, disponibilidade
- Restrições: budget, equipe, prazo
- Perguntas-chave: quantos usuários? quantos requests/segundo? tamanho dos dados?

**2. Estimativas de escala (5 min)**

- QPS (queries per second): diário → por segundo
- Storage: tamanho médio × número de registros × retenção
- Bandwidth: QPS × tamanho médio da resposta
- Exemplo: 100M DAU, 10 requests/dia = ~12K QPS

**3. API Design (5 min)**

- Definir os endpoints principais
- Request/Response format
- Autenticação e rate limiting

**4. Diagrama de alto nível (10 min)**

- Clients → Load Balancer → API Servers → Database
- Cache layer, CDN, message queues onde necessário

**5. Deep dive em componentes (15-20 min)**

- Foco nos componentes mais complexos ou interessantes
- Database schema, indexação, particionamento
- Caching strategy, invalidação
- Escalabilidade: horizontal vs vertical

**6. Trade-offs e melhorias (5 min)**

- Bottlenecks e como resolver
- Monitoramento e alertas
- Evolução futura

### Conceitos fundamentais

#### Escalabilidade

- **Vertical (scale up):** mais CPU/RAM na mesma máquina. Simples, mas tem limite.
- **Horizontal (scale out):** mais máquinas. Requer stateless services e load balancing.

#### Load Balancing

Distribui tráfego entre servidores. Algoritmos:

- **Round Robin:** distribui sequencialmente. Simples, bom para servidores iguais.
- **Least Connections:** envia para o servidor com menos conexões ativas.
- **Consistent Hashing:** mapeia requests para servidores com mínima redistribuição quando servidores entram/saem.
- **Ferramentas:** Nginx, HAProxy, AWS ALB/NLB.

#### Caching

Armazenar dados frequentemente acessados em memória para reduzir latência.

- **Client-side:** browser cache, CDN
- **Server-side:** Redis, Memcached
- **Database:** query cache, materialized views
- **Strategies:** Cache-aside (lazy loading), Write-through, Write-behind
- **Invalidação:** TTL, event-based, versioned keys

#### Database

- **SQL vs NoSQL:** consistência vs flexibilidade
- **Sharding:** dividir dados em partições. Por hash (distribuição uniforme) ou range (queries de range).
- **Replication:** primary-replica para leitura, multi-primary para escrita distribuída.
- **CAP Theorem:** Consistency, Availability, Partition tolerance — escolha 2 (na prática, AP ou CP).

#### Message Queues

Desacoplam produtores de consumidores. Garantem processamento assíncrono e resiliência.

- **Kafka:** alto throughput, ordenação por partição, retenção de mensagens.
- **RabbitMQ:** roteamento flexível, filas com prioridade.
- **SQS:** managed, simples, integrado com AWS.

#### CDN (Content Delivery Network)

Distribui conteúdo estático em servidores geograficamente próximos dos usuários. Reduz latência e carga no servidor principal.

### Sistemas clássicos de entrevista

| Sistema       | Conceitos-chave                                       |
| ------------- | ----------------------------------------------------- |
| URL Shortener | Hash, base62, banco KV, cache, redirect 301/302       |
| Twitter Feed  | Fan-out on write vs read, timeline cache, pub/sub     |
| Chat System   | WebSocket, presence service, message queue, ordenação |
| Rate Limiter  | Token bucket, sliding window, distributed counter     |
| Notification  | Multi-channel, template engine, priority queue, retry |
| File Storage  | Chunking, dedup, metadata DB, object storage (S3)     |

## Quando usar

- **Em toda entrevista de senior+:** system design é esperado
- **No dia a dia:** ao planejar novos features, migrações, ou evoluções de arquitetura
- **Em ADRs:** documentar decisões de design com trade-offs

## Armadilhas comuns

- **Pular clarificação de requisitos:** sem saber a escala, você pode over-engineer ou under-engineer.
- **Projetar para Google-scale:** a maioria dos sistemas não precisa de sharding. Comece simples.
- **Não falar sobre trade-offs:** todo design tem compromissos. Explicitá-los demonstra senioridade.
- **Ignorar não-funcionais:** disponibilidade, latência, consistência são tão importantes quanto features.
- **Monólogo:** system design é uma conversa. Pergunte, valide, ajuste.

## Na prática (da minha experiência)

> No MedEspecialista, projetei a evolução de um monolito para microserviços. O sistema de agendamentos usa PostgreSQL com sharding por clínica, cache Redis para slots disponíveis (TTL de 5 minutos), e Kafka para notificações assíncronas. A decisão de não usar eventual consistency no core de agendamentos (evitando double-booking) mas aceitar eventual consistency nas notificações foi um trade-off importante que melhorou performance sem comprometer a integridade dos dados.

## How to explain in English

"In system design interviews, I follow a structured approach. I start by clarifying requirements — both functional and non-functional — because the scale and constraints drive every subsequent decision.

For example, if asked to design a URL shortener, I'd first establish the scale: how many URLs per day? What's the read-to-write ratio? Do we need analytics? Then I'd walk through the architecture: a hash function to generate short codes, a key-value store for the mapping, a cache layer for hot URLs, and a CDN for the redirect endpoints.

What I focus on is trade-offs. For the URL shortener, I'd discuss: do we use base62 encoding or MD5 hashing? Base62 is shorter and predictable, but MD5 avoids sequential IDs. Do we pre-generate codes or generate on-demand? Pre-generation avoids collisions but wastes storage. These are the kinds of decisions that show engineering judgment.

In my experience building production systems, I've learned that the most important principle is to start simple and scale incrementally. A well-designed monolith with clear boundaries can handle surprising scale, and premature distributed architecture creates more problems than it solves."

### Key vocabulary

- escalabilidade horizontal → horizontal scaling / scale out
- balanceamento de carga → load balancing
- fragmentação de dados → sharding / data partitioning
- replicação → replication
- consistência eventual → eventual consistency
- taxa de requisições → requests per second (RPS) / queries per second (QPS)
- ponto único de falha → single point of failure (SPOF)
- tolerância a falhas → fault tolerance
- throughput → throughput: volume processado por unidade de tempo
- latência → latency: tempo de resposta

## Recursos

> [!info] SYSTEM DESIGN: ALÉM DA ENTREVISTA
> [https://www.youtube.com/live/-8tdjn30SSw?si=kcvd_nTLIYMNIrM6](https://www.youtube.com/live/-8tdjn30SSw?si=kcvd_nTLIYMNIrM6)

> [!info] 18 System Design Concepts Every Engineer Must Know
> [https://www.designgurus.io/blog/system-design-interview-fundamentals](https://www.designgurus.io/blog/system-design-interview-fundamentals)

> [!info] Load Balancing Algorithms
> [https://dev.to/somadevtoo/system-design-basics-load-balancing-algorithms-2559](https://dev.to/somadevtoo/system-design-basics-load-balancing-algorithms-2559)

- [System Design Primer (GitHub)](https://github.com/donnemartin/system-design-primer) — referência completa
- [Designing Data-Intensive Applications](https://dataintensive.net/) — Martin Kleppmann (livro essencial)
- [Architectural Katas](https://www.architecturalkatas.com/) — exercícios práticos

## Veja também

- [[API Design]]
- [[Banco de dados]]
- [[Kafka]]
- [[Arquitetura de Software]]
- [[System Design Practice]]
