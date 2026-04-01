---
title: "Arquitetura de Software"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - arquitetura
  - entrevista
publish: false
---

# Arquitetura de Software

Decisões estruturais que definem como um sistema é organizado — estilos, padrões, princípios e documentação.

## O que é

Arquitetura de Software é o conjunto de decisões de alto nível sobre a estrutura de um sistema: como componentes se organizam, comunicam e evoluem. Não existe arquitetura "certa" — existe a mais adequada ao contexto (equipe, escala, prazo, domínio).

## Estilos arquiteturais

### Clean Architecture

Proposta por Uncle Bob, organiza o código em camadas concêntricas onde dependências apontam para dentro (Dependency Rule):

```text
        ┌─────────────────────────┐
        │    Frameworks & Drivers  │  ← UI, DB, APIs externas
        │  ┌───────────────────┐  │
        │  │    Adapters        │  │  ← Controllers, Gateways, Presenters
        │  │  ┌─────────────┐  │  │
        │  │  │  Use Cases   │  │  │  ← Application Business Rules
        │  │  │  ┌────────┐  │  │  │
        │  │  │  │Entities │  │  │  │  ← Enterprise Business Rules
        │  │  │  └────────┘  │  │  │
        │  │  └─────────────┘  │  │
        │  └───────────────────┘  │
        └─────────────────────────┘
```

**Regra fundamental:** camadas internas não conhecem camadas externas. Entities não sabem que banco de dados existe.

**Benefícios:** domínio isolado, testável sem infraestrutura, independente de framework.

**Trade-off:** mais camadas = mais indireção. Para CRUD simples, pode ser over-engineering.

### Arquitetura Hexagonal (Ports & Adapters)

Proposta por Alistair Cockburn. O core da aplicação define **Ports** (interfaces) e o mundo externo implementa **Adapters**:

```text
                    ┌──────────────┐
    HTTP Adapter ──→│  Port (in)   │
    CLI Adapter  ──→│              │
                    │   Domain     │
                    │   Logic      │
    DB Adapter   ←──│  Port (out)  │
    Email Adapter←──│              │
                    └──────────────┘
```

- **Driving Ports (entrada):** interfaces que o mundo externo usa para acionar a aplicação (API, CLI, eventos)
- **Driven Ports (saída):** interfaces que a aplicação usa para acessar o mundo externo (banco, email, APIs)
- **Adapters:** implementações concretas dos ports

**Na prática:** em Spring Boot, um `@Service` que depende de uma interface `PatientRepository` (port) implementada por `JpaPatientRepository` (adapter).

### Como tudo se conecta (Explicit Architecture)

Segundo Herberto Graça, Clean, Hexagonal, Onion e DDD convergem:

- **Interface do Usuário** (delivery mechanisms) → controllers, CLI, event listeners
- **Application Core** (lógica de negócio) → use cases, domain services, entities
- **Infrastructure** (ferramentas externas) → banco, email, APIs, filas

Todas as dependências apontam para o center (domain). O domain nunca importa infrastructure. CQRS separa Commands (escrita) de Queries (leitura) para otimizar cada lado independentemente.

### Tomato Architecture

Abordagem pragmática para projetos que não precisam da cerimônia completa de Clean/Hexagonal:

- Organização por **feature/domínio** (package by feature), não por camada
- Separação clara de business logic e infrastructure
- Menos overhead — ideal para equipes pequenas e projetos que precisam evoluir rápido
- Quando a complexidade crescer, refatorar para Hexagonal é natural

**Quando usar:** projetos médios onde Clean Architecture seria over-engineering, mas "jogar tudo no controller" seria under-engineering.

## Princípios

### SOLID

Ver [[Design Patterns]] e [[Orientação a Objetos]] para detalhes completos. Em resumo:

| Princípio | Essência |
| --- | --- |
| Single Responsibility | Uma razão para mudar |
| Open/Closed | Extender sem modificar |
| Liskov Substitution | Subtipos são substituíveis |
| Interface Segregation | Interfaces pequenas e focadas |
| Dependency Inversion | Dependa de abstrações |

### 12 Fatores (Twelve-Factor App)

Metodologia para construir aplicações SaaS modernas. Os mais relevantes para entrevistas:

1. **Codebase:** um repo, muitos deploys (staging, prod)
2. **Dependencies:** declare e isole (Maven, npm, Docker)
3. **Config:** configuração via variáveis de ambiente, nunca no código
4. **Backing services:** trate banco, cache, filas como recursos anexáveis
5. **Build, release, run:** etapas separadas e imutáveis
6. **Processes:** stateless — estado vai em backing services (Redis, DB)
7. **Port binding:** exporte serviço via porta (embedded server)
8. **Concurrency:** escale por processos (horizontal scaling)
9. **Disposability:** start rápido, shutdown graceful
10. **Dev/prod parity:** ambientes o mais similares possível (Docker!)
11. **Logs:** trate como event streams (stdout → coletado externamente)
12. **Admin processes:** tarefas pontuais rodam como processos (migrations, scripts)

### Clean Code

Código que é fácil de ler, entender e modificar:

- **Nomes significativos:** variáveis e funções autoexplicativas
- **Funções pequenas:** fazem uma coisa só
- **DRY (Don't Repeat Yourself):** evitar duplicação de lógica
- **KISS (Keep It Simple):** a solução mais simples que funciona
- **Boy Scout Rule:** deixe o código melhor do que encontrou

## DDD (Domain-Driven Design)

Abordagem que coloca o domínio do negócio no centro do design:

### Conceitos táticos

- **Entity:** objeto com identidade (Patient, Order)
- **Value Object:** objeto definido por seus atributos, sem identidade (Money, Address)
- **Aggregate:** cluster de entidades com uma raiz (Order → OrderItems)
- **Repository:** abstração para persistência de aggregates
- **Domain Service:** lógica que não pertence a nenhuma entidade
- **Domain Event:** fato que aconteceu no domínio ("OrderPlaced")

### Conceitos estratégicos

- **Bounded Context:** limite onde um modelo é consistente. Cada microserviço geralmente é um bounded context.
- **Ubiquitous Language:** linguagem compartilhada entre devs e negócio
- **Context Map:** como bounded contexts se relacionam

**Na prática:** DDD é valioso para domínios complexos. Para CRUD simples, é overhead. O [[Event Storming]] é uma técnica para descobrir bounded contexts.

## Microserviços

Estilo arquitetural onde a aplicação é uma coleção de serviços pequenos, independentemente deployáveis e loosely coupled.

### Quando migrar de monolito

- Equipe cresceu e o monolito virou gargalo
- Partes do sistema precisam escalar independentemente
- Deploy do monolito ficou lento ou arriscado
- **Não migre prematuramente:** um monolito modular bem feito é melhor que microserviços mal feitos

### Padrões essenciais

| Padrão | O que resolve |
| --- | --- |
| API Gateway | Ponto de entrada único, routing, auth |
| Service Discovery | Encontrar serviços dinamicamente |
| Circuit Breaker | Evitar cascata de falhas |
| Saga | Transações distribuídas sem 2PC |
| Strangler Fig | Migração gradual de monolito |
| Sidecar | Cross-cutting concerns (logging, tracing) |

### Anti-patterns

- **Distributed Monolith:** microserviços acoplados que precisam ser deployados juntos
- **Shared Database:** vários serviços acessando o mesmo banco
- **Chatty Services:** muitas chamadas síncronas entre serviços

## Documentação de Arquitetura

### C4 Model

Abordagem hierárquica para diagramar arquitetura em 4 níveis:

1. **System Context:** o sistema e suas interações externas (para stakeholders)
2. **Container:** componentes técnicos — apps, bancos, filas (para arquitetos)
3. **Component:** estrutura interna de cada container (para devs)
4. **Code:** classes e funções (raramente necessário)

**Regra:** comece pelo contexto (mais abstrato) e desça conforme a necessidade de detalhe.

### ADRs (Architectural Decision Records)

Documentar decisões arquiteturais com: contexto, decisão, consequências e status. Versionados junto ao código.

```markdown
# ADR-001: Usar PostgreSQL como banco principal

## Status: Accepted

## Contexto
Precisamos de banco relacional com suporte a JSONB, 
full-text search, e maturidade operacional.

## Decisão
PostgreSQL 16 como banco principal.

## Consequências
+ Suporte nativo a JSONB reduz necessidade de MongoDB
+ Ecossistema maduro de ferramentas
- Equipe precisa aprender features específicas do PG
```

## Observabilidade

Os três pilares para entender o que acontece em produção:

- **Logs:** eventos discretos (o que aconteceu). Structured logging (JSON) para facilitar busca.
- **Metrics:** números ao longo do tempo (RED: Rate, Errors, Duration). Prometheus + Grafana.
- **Traces:** caminho de uma requisição através de múltiplos serviços. OpenTelemetry.

**Na prática:** Spring Boot Actuator + Micrometer exportam métricas. OpenTelemetry coleta traces distribuídos. Logs estruturados com correlation IDs permitem rastrear uma requisição do frontend ao banco.

## Armadilhas comuns

- **Astronaut Architecture:** abstrações demais, código de menos. Ship first, refactor later.
- **Resume-Driven Development:** escolher tecnologia pra enfeitar o currículo, não pra resolver o problema.
- **Premature Distribution:** microserviços antes de ter um monolito modular funcionando.
- **Ignorar o contexto:** a "melhor" arquitetura depende da equipe, escala, prazo e domínio.

## Na prática (da minha experiência)

> Na Muvz, apliquei Hexagonal Architecture com DDD para migrar um monolito Java EJB para 5 microserviços Spring Boot. Usei Event Storming para mapear bounded contexts e definir os limites dos serviços. Kafka como event broker entre eles. A decisão de não usar Clean Architecture "pura" foi pragmática — a equipe era pequena e a cerimônia completa reduziria a velocidade sem benefício proporcional. No MedEspecialista, comecei a reescrita do backend Node.js usando Clean Architecture (Domain/Application/Infrastructure) como preparação para futura migração para NestJS + TypeScript.

## How to explain in English

"Software architecture is about making structural decisions that are hard to change later. My approach is pragmatic: I choose the simplest architecture that meets the current requirements, with clear boundaries that allow evolution.

For most projects, I start with a modular monolith using Hexagonal Architecture principles — the domain logic depends on interfaces (ports), and infrastructure provides the implementations (adapters). This gives me the isolation benefits of Clean Architecture without the overhead of strict layering.

When I need to split into microservices, I use Domain-Driven Design to identify bounded contexts through Event Storming workshops. Each microservice owns its data and communicates through events (Kafka) or APIs. I follow the Strangler Fig pattern for gradual migration rather than big-bang rewrites.

For documentation, I use C4 diagrams to communicate at the right level of abstraction and ADRs to record architectural decisions with their context and trade-offs. This creates a searchable history of why the system is shaped the way it is."

### Key vocabulary

- estilo arquitetural → architectural style
- arquitetura hexagonal → hexagonal architecture / ports and adapters
- contexto delimitado → bounded context (DDD)
- linguagem ubíqua → ubiquitous language
- registro de decisão arquitetural → architectural decision record (ADR)
- observabilidade → observability
- monolito modular → modular monolith
- padrão estrangulador → strangler fig pattern

## Recursos

### Cursos em vídeo

> [!info] Fundamentos de Arquitetura de Software
> [https://www.youtube.com/playlist?list=PLkpjQs-GfEMPzOzinFrqfkkfZy2DpwpBh](https://www.youtube.com/playlist?list=PLkpjQs-GfEMPzOzinFrqfkkfZy2DpwpBh)

> [!info] Arquitetura de Software - Primeira Temporada
> [https://www.youtube.com/playlist?list=PLkpjQs-GfEMNcWDlIck2I5TGBSSRCK39L](https://www.youtube.com/playlist?list=PLkpjQs-GfEMNcWDlIck2I5TGBSSRCK39L)

> [!info] Padrões Arquiteturais
> [https://www.youtube.com/playlist?list=PLkpjQs-GfEMMoh78fnnHtrhK1iWe-ZSJ5](https://www.youtube.com/playlist?list=PLkpjQs-GfEMMoh78fnnHtrhK1iWe-ZSJ5)

> [!info] Escalabilidade e Performance
> [https://www.youtube.com/playlist?list=PLkpjQs-GfEMOqjfVgNktbKoJ7dcEiuYyl](https://www.youtube.com/playlist?list=PLkpjQs-GfEMOqjfVgNktbKoJ7dcEiuYyl)

> [!info] DDD do jeito certo
> [https://www.youtube.com/playlist?list=PLkpjQs-GfEMN8CHp7tIQqg6JFowrIX9ve](https://www.youtube.com/playlist?list=PLkpjQs-GfEMN8CHp7tIQqg6JFowrIX9ve)

> [!info] SOLID e DDD na prática
> [https://www.youtube.com/live/oKpZvWWning?si=DXKlNFNjNgYuowx1](https://www.youtube.com/live/oKpZvWWning?si=DXKlNFNjNgYuowx1)

> [!info] A Forma Ideal de Projetos Web - Os 12 Fatores
> [https://youtu.be/gpJgtED36U4?si=W3QxIMU54P2xiF_p](https://youtu.be/gpJgtED36U4?si=W3QxIMU54P2xiF_p)

> [!info] SOLID fica FÁCIL com Essas Ilustrações
> [https://www.youtube.com/watch?v=6SfrO3D4dHM](https://www.youtube.com/watch?v=6SfrO3D4dHM)

> [!info] CQS: encapsulamento em POO
> [https://youtu.be/7NVbDGIMHjo?si=M2eGO4kZr0Vzs3uU](https://youtu.be/7NVbDGIMHjo?si=M2eGO4kZr0Vzs3uU)

> [!info] RETRY - Padrões de resiliência para Microsserviços
> [https://www.youtube.com/watch?v=1MkPpKPyBps](https://www.youtube.com/watch?v=1MkPpKPyBps)

### Artigos e referências

- [Architecture Antipatterns](https://architecture-antipatterns.tech/)
- [Google Engineering Practices](https://google.github.io/eng-practices/)
- [The S.O.L.I.D Principles in Pictures](https://medium.com/backticks-tildes/the-s-o-l-i-d-principles-in-pictures-b34ce2f1e898)
- [Microservices — Martin Fowler](https://martinfowler.com/articles/microservices.html)
- [Microservices.io](https://microservices.io/index.html) — linguagem de padrões
- [What do you mean by "Event-Driven"? — Martin Fowler](https://martinfowler.com/articles/201701-event-driven.html)
- [DDD, Hexagonal, Onion, Clean, CQRS — How I put it all together](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/)
- [The Software Architecture Chronicles](https://herbertograca.com/2017/07/03/the-software-architecture-chronicles/)
- [Documenting Software Architecture](https://herbertograca.com/2019/08/12/documenting-software-architecture/)
- [Exponential Backoff and Jitter — AWS](https://aws.amazon.com/pt/blogs/architecture/exponential-backoff-and-jitter/)

### Clean Architecture

- [Clean Coder Blog — The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Descomplicando a Clean Architecture](https://medium.com/luizalabs/descomplicando-a-clean-architecture-cf4dfc4a1ac6)
- [Clean Architecture: descubra o que é e onde aplicar](https://www.zup.com.br/blog/clean-architecture-arquitetura-limpa)

### Hexagonal Architecture

- [Hexagonal Architecture — There Are Always Two Sides](https://medium.com/ssense-tech/hexagonal-architecture-there-are-always-two-sides-to-every-story-bc0780ed7d9c)
- [How to Implement a Hexagonal Architecture](https://www.freecodecamp.org/news/implementing-a-hexagonal-architecture/)

### Tomato Architecture

- [Tomato Architecture — A Pragmatic Approach](https://www.sivalabs.in/tomato-architecture-pragmatic-approach-to-software-design/)
- [Package by Feature](https://phauer.com/2020/package-by-feature/)
- [Demo — Spring Boot](https://github.com/sivaprasadreddy/tomato-architecture-spring-boot-demo)

### C4 Model e Documentação

- [C4 Model](https://c4model.com/)
- [arc42 Documentation](https://arc42.org/documentation/)
- [ADR GitHub Organization](https://adr.github.io/)

> [!info] Curso de C4 Model na prática
> [https://www.youtube.com/playlist?list=PLxuFqIk29JL0d_ESgZomFSEOzywPMmAqy](https://www.youtube.com/playlist?list=PLxuFqIk29JL0d_ESgZomFSEOzywPMmAqy)

### Ferramentas

- [LocalStack](https://github.com/localstack/localstack) — simula AWS localmente
- [OpenTelemetry](https://opentelemetry.io/) — observabilidade unificada
- [Technology Radar — Thoughtworks](https://www.thoughtworks.com/en-br/radar) — tendências tecnológicas
- [Architectural Katas](https://www.architecturalkatas.com/) — exercícios práticos

## Veja também

- [[System Design]]
- [[Design Patterns]]
- [[API Design]]
- [[Orientação a Objetos]]
- [[Event Storming]]
- [[Mensageria]]
- [[Event Streaming]]
- [[Kafka]]
