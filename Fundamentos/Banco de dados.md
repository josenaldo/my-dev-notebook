---
title: "Banco de dados"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - fundamentos
  - entrevista
publish: false
---

# Banco de dados

Sistemas para armazenar, consultar e manipular dados de forma estruturada e eficiente.

## O que é

Bancos de dados são a base de quase todo sistema. Em entrevistas de senior, espera-se domínio de modelagem relacional, entendimento de NoSQL, indexação, transações (ACID) e decisões de trade-off entre consistência e performance.

## Como funciona

### Bancos Relacionais (SQL)

Organizam dados em tabelas com relações definidas por chaves estrangeiras. Garantem ACID:

- **Atomicity:** transação completa ou não acontece
- **Consistency:** dados sempre em estado válido
- **Isolation:** transações concorrentes não interferem
- **Durability:** dados commitados sobrevivem a falhas

**Principais:** PostgreSQL, MySQL, Oracle, SQL Server.

### Bancos NoSQL

| Tipo | Exemplo | Use case |
| --- | --- | --- |
| Documento | MongoDB, CouchDB | Schemas flexíveis, dados semi-estruturados |
| Chave-valor | Redis, DynamoDB | Cache, sessões, dados simples e rápidos |
| Coluna larga | Cassandra, HBase | Escrita massiva, séries temporais |
| Grafo | Neo4j | Relações complexas (redes sociais, recomendações) |

### Índices

Estruturas que aceleram consultas ao custo de espaço e escrita mais lenta.

- **B-Tree:** default na maioria dos bancos. Bom para range queries e equality.
- **Hash Index:** O(1) para equality, não suporta range.
- **GIN/GiST (PostgreSQL):** para full-text search, JSONB, arrays.
- **Índice composto:** múltiplas colunas. A ordem importa — segue a regra do "leftmost prefix".

**Regra prática:** indexe colunas usadas em WHERE, JOIN e ORDER BY. Mas cada índice tem custo de escrita.

### Modelagem

- **Normalização:** eliminar redundância (1NF → 2NF → 3NF → BCNF). Melhora consistência, mas pode prejudicar performance de leitura.
- **Desnormalização:** adicionar redundância proposital para acelerar leituras. Comum em sistemas de leitura intensiva.
- **Trade-off:** normalizado para escrita/consistência, desnormalizado para leitura/performance.

### Transações e Isolamento

| Nível | Dirty Read | Non-repeatable Read | Phantom Read |
| --- | --- | --- | --- |
| READ UNCOMMITTED | Sim | Sim | Sim |
| READ COMMITTED | Não | Sim | Sim |
| REPEATABLE READ | Não | Não | Sim |
| SERIALIZABLE | Não | Não | Não |

**Na prática:** READ COMMITTED é o default do PostgreSQL. REPEATABLE READ é o default do MySQL/InnoDB.

### IDs: UUID vs ULID vs Auto-increment

- **Auto-increment:** simples, ordenável, mas expõe volume de dados e não funciona em sistemas distribuídos.
- **UUID v4:** aleatório, bom para distribuído, mas ruim para índices B-Tree (fragmentação).
- **ULID:** ordenável por tempo + aleatório. Melhor performance em índices que UUID. Boa escolha para sistemas distribuídos.

## Quando usar

- **SQL (PostgreSQL):** default para aplicações com dados relacionais, transações complexas, integridade referencial
- **Redis:** cache, sessões, rate limiting, pub/sub simples
- **MongoDB:** prototipagem rápida, schemas que mudam muito, dados de documento
- **Elasticsearch:** full-text search, logs, analytics
- **Cassandra:** escrita massiva, dados de séries temporais, alta disponibilidade

## Armadilhas comuns

- **N+1 queries:** carregar entidades uma a uma dentro de um loop. Resolver com JOIN, subquery ou batch loading (JPA `@EntityGraph`).
- **Indexar tudo:** cada índice consome espaço e torna escritas mais lentas. Indexe com base em queries reais.
- **Ignorar plano de execução:** use EXPLAIN ANALYZE para entender se o banco está usando seus índices.
- **Não pensar em migração:** mudanças de schema precisam ser backward-compatible em produção. Blue-green migrations.
- **UUID como PK sem pensar:** UUID v4 fragmenta índices B-Tree. Considere ULID ou UUID v7.

## Na prática (da minha experiência)

> Em projetos com Spring Boot e JPA, uso PostgreSQL como banco principal com Flyway para migrações versionadas. No MedEspecialista, modelei o sistema de agendamentos com normalização 3NF para manter consistência nos dados médicos, mas criei views materializadas para consultas de relatórios que precisavam de performance. Para IDs, migrei de auto-increment para ULID quando o sistema começou a precisar de integração entre microserviços.

## How to explain in English

"Database design is one of the areas where a senior developer adds the most value. I start by understanding the access patterns — who reads, who writes, how often, and what consistency guarantees are needed.

For most applications, I default to PostgreSQL. It's robust, has excellent indexing capabilities including GIN indexes for JSONB, and handles transactions well with MVCC. I use Flyway for schema migrations, which keeps database changes versioned alongside the application code.

When it comes to indexing, I follow a data-driven approach. I use EXPLAIN ANALYZE to understand query plans before adding indexes. A common mistake is over-indexing — every index speeds up reads but slows down writes. I focus on the queries that matter most, typically the ones in hot paths.

For distributed systems, I've moved from auto-increment IDs to ULIDs. They're globally unique, sortable by time, and don't fragment B-Tree indexes the way random UUIDs do. This was an important decision when we started splitting our monolith into microservices."

### Key vocabulary

- chave primária → primary key (PK)
- chave estrangeira → foreign key (FK)
- índice → index: estrutura que acelera consultas
- normalização → normalization: eliminar redundância
- transação → transaction: unidade atômica de trabalho
- isolamento → isolation level: controle de concorrência
- plano de execução → query execution plan / EXPLAIN
- migração → migration: mudança versionada de schema

## Recursos

### Cursos

> [!info] Curso de Modelagem de Dados
> Curso completo de Modelagem de Bancos de Dados relacionais.
> [https://www.youtube.com/playlist?list=PLucm8g_ezqNoNHU8tjVeHmRGBFnjDIlxD](https://www.youtube.com/playlist?list=PLucm8g_ezqNoNHU8tjVeHmRGBFnjDIlxD)

> [!info] Curso de Modelagem Relacional - Banco de Dados
> [https://www.youtube.com/playlist?list=PLg5-aZqPjMmAo-kX-1l6BQS_yIIObGc3C](https://www.youtube.com/playlist?list=PLg5-aZqPjMmAo-kX-1l6BQS_yIIObGc3C)

> [!info] Álgebra Relacional
> Fundamentos da Álgebra Relacional, base da Linguagem SQL.
> [https://www.youtube.com/playlist?list=PLdoTFRH60cIDTo_-mZvnuwg_uJeMbu6ZP](https://www.youtube.com/playlist?list=PLdoTFRH60cIDTo_-mZvnuwg_uJeMbu6ZP)

> [!info] Banco de Dados (2020.1)
> [https://www.youtube.com/playlist?list=PLxI8Can9yAHeZcEzZElhxwsQTf9MaG6sS](https://www.youtube.com/playlist?list=PLxI8Can9yAHeZcEzZElhxwsQTf9MaG6sS)

> [!info] Curso de SQL com MySQL (Completo)
> [https://www.youtube.com/playlist?list=PLbIBj8vQhvm2WT-pjGS5x7zUzmh4VgvRk](https://www.youtube.com/playlist?list=PLbIBj8vQhvm2WT-pjGS5x7zUzmh4VgvRk)

### Ferramentas

- [Joins visualizer](https://joins.spathon.com/) — visualização interativa de JOINs
- [Joins explicados (vídeo)](https://youtu.be/zGSv0VaOtR0?si=kel-SKe58YTxtIO6)

### Artigos

> [!info] UUIDs são ruins? Entenda o que é ULID
> [https://blog.lsantos.dev/o-que-e-ulid/](https://blog.lsantos.dev/o-que-e-ulid/)

> [!info] Não use UUID como PK nas tabelas do seu banco de dados
> [https://gist.github.com/rponte/bf362945a1af948aa04b587f8ff332f8](https://gist.github.com/rponte/bf362945a1af948aa04b587f8ff332f8)

## Veja também

- [[System Design]]
- [[API Design]]
- [[Java Fundamentals]] — JPA, JDBC
