---
title: "Design Patterns"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - arquitetura
  - entrevista
publish: false
---

# Design Patterns

Soluções reutilizáveis para problemas recorrentes em design de software.

## O que é

Design Patterns são soluções catalogadas (GoF — Gang of Four) para problemas comuns. Em entrevistas, não se espera que você memorize todos os 23 padrões — mas que conheça os mais usados e saiba quando (e quando **não**) aplicar.

## Como funciona

### Padrões Criacionais

**Singleton:** uma única instância global. Em Spring, beans são singleton por padrão (`@Service`, `@Component`).

**Factory Method:** delega a criação de objetos a subclasses. Útil quando o tipo exato do objeto depende de contexto.

```java
// Em vez de:
Notification n = new EmailNotification();

// Factory:
Notification n = NotificationFactory.create(type);
```

**Builder:** constrói objetos complexos passo a passo. Útil quando o construtor teria muitos parâmetros.

```java
var user = User.builder()
    .name("Josenaldo")
    .email("josenaldo@email.com")
    .role(Role.ADMIN)
    .build();
```

### Padrões Estruturais

**Adapter:** converte a interface de uma classe para outra esperada pelo cliente. Útil para integrar com APIs de terceiros.

**Decorator:** adiciona comportamento a um objeto sem alterar sua classe. Em Java: `BufferedReader` decorando `FileReader`. Em TS: decorators do NestJS.

**Facade:** interface simplificada para um subsistema complexo. Exemplo: um `PaymentService` que orquestra validação, cobrança, notificação e logging.

### Padrões Comportamentais

**Strategy:** encapsula algoritmos intercambiáveis. O cliente escolhe qual usar em runtime.

```java
interface SortStrategy { void sort(List<?> data); }

class ReportService {
    private final SortStrategy strategy;
    // O algoritmo de ordenação pode mudar sem alterar ReportService
}
```

**Observer:** notifica dependentes quando o estado muda. Base de sistemas event-driven. Em JS: `EventEmitter`. Em Spring: `ApplicationEvent`.

**Template Method:** define o esqueleto de um algoritmo, delegando passos para subclasses.

**Command:** encapsula uma requisição como objeto. Útil para undo/redo, filas de trabalho.

### Padrões no mundo real

| Padrão | Onde você já usa | Framework |
| --- | --- | --- |
| Singleton | Beans do Spring | Spring IoC |
| Observer | Event listeners | Spring Events, Node EventEmitter |
| Strategy | Algoritmos intercambiáveis | Comparator em Java, sort callbacks em JS |
| Builder | Construção de objetos complexos | Lombok @Builder, StringBuilder |
| Decorator | Middleware, interceptors | Express middleware, Spring AOP |
| Facade | Service layer | Spring @Service orquestrando repositories |
| Factory | Criação condicional | Spring FactoryBean, Abstract Factory |
| Proxy | Lazy loading, caching | JPA proxies, Spring AOP proxies |
| Repository | Acesso a dados | Spring Data JPA |

## Quando usar

- **Strategy:** quando tem múltiplos algoritmos para a mesma operação (formas de pagamento, tipos de exportação)
- **Observer:** quando uma ação precisa disparar várias reações desacopladas
- **Builder:** quando o construtor tem 4+ parâmetros ou parâmetros opcionais
- **Factory:** quando a criação do objeto envolve lógica condicional
- **Adapter:** quando precisa integrar com código legado ou API de terceiros

## Armadilhas comuns

- **Pattern mania:** aplicar padrões onde um `if` resolveria. Padrões adicionam complexidade — use quando o problema justifica.
- **Singleton mutável:** estado global compartilhado causa bugs difíceis de debugar e torna testes frágeis.
- **Observer sem cleanup:** memory leaks quando listeners não são removidos.
- **Abstract Factory prematura:** criar fábricas de fábricas sem necessidade real.
- **Confundir padrão com framework:** Spring já implementa muitos padrões internamente. Não reimplemente o que o framework faz.

## Na prática (da minha experiência)

> No dia a dia com Spring Boot, uso Strategy para implementar diferentes formas de processamento — por exemplo, diferentes estratégias de cálculo de frete dependendo da região. O Spring injeta todas as implementações de uma interface via `List<Strategy>`, e um resolver escolhe qual usar. Builder é onipresente com Lombok. Observer via `@EventListener` para desacoplar ações como envio de email após cadastro de paciente.

## How to explain in English

"Design patterns are solutions I use daily, often through framework abstractions. In Spring Boot, the IoC container itself is a massive application of Dependency Injection and Singleton. When I create a `@Service` class that depends on interfaces, I'm applying Strategy through constructor injection.

The patterns I use most frequently are Strategy for interchangeable algorithms, Observer for event-driven decoupling, Builder for complex object construction, and Facade for simplifying complex subsystems into clean service interfaces.

I'm careful not to over-apply patterns. A common anti-pattern I've seen is creating a Strategy interface with only one implementation — that's premature abstraction. I apply patterns when I have a concrete need, not as a prophylactic measure. Three similar `if` statements are better than a premature abstraction that makes the code harder to follow."

### Key vocabulary

- padrão de projeto → design pattern
- padrão criacional → creational pattern
- padrão estrutural → structural pattern
- padrão comportamental → behavioral pattern
- princípio da responsabilidade única → single responsibility principle
- inversão de controle → inversion of control (IoC)
- injeção de dependência → dependency injection (DI)
- desacoplamento → decoupling / loose coupling

## Recursos

- [Refactoring Guru — Design Patterns](https://refactoring.guru/design-patterns) — catálogo visual com exemplos
- [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/0596007124/) — livro clássico acessível
- [Source Making](https://sourcemaking.com/design_patterns) — exemplos práticos

## Veja também

- [[Orientação a Objetos]]
- [[Arquitetura de Software]]
- [[Spring Boot]]
