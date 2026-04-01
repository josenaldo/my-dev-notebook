---
title: "Orientação a Objetos"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - fundamentos
  - entrevista
publish: false
---

# Orientação a Objetos

Paradigma que organiza software em objetos que encapsulam dados e comportamento.

## O que é

Orientação a Objetos (OOP) é um paradigma de programação baseado em quatro pilares: encapsulamento, herança, polimorfismo e abstração. Em entrevistas, espera-se que um senior não apenas defina esses conceitos, mas demonstre como aplica (e quando **não** aplica) cada um.

## Como funciona

### Os 4 pilares

**Encapsulamento:** esconder detalhes internos e expor apenas uma interface pública. Em Java: modificadores de acesso (`private`, `protected`, `public`). Em TS: `private`, `readonly`.

**Herança:** uma classe herda comportamento de outra. Útil para hierarquias naturais, mas perigoso quando abusado — prefira composição sobre herança.

**Polimorfismo:** objetos de tipos diferentes respondem à mesma interface. Em Java: interfaces e classes abstratas. Em TS: interfaces e types. Polimorfismo de runtime (override) vs compile-time (overload).

**Abstração:** modelar conceitos do domínio sem expor a implementação. Uma interface `PaymentGateway` abstrai se o pagamento vai por Stripe, PagSeguro ou PIX.

### SOLID

| Princípio | Nome | Resumo |
|-----------|------|--------|
| S | Single Responsibility | Uma classe, uma razão para mudar |
| O | Open/Closed | Aberto para extensão, fechado para modificação |
| L | Liskov Substitution | Subtipos devem ser substituíveis pelo tipo base |
| I | Interface Segregation | Interfaces pequenas e específicas |
| D | Dependency Inversion | Dependa de abstrações, não de implementações concretas |

### Composição sobre Herança

Herança cria acoplamento forte entre classes. Composição (injetar dependências) é mais flexível:

```typescript
// Herança (frágil)
class EmailNotifier extends Notifier { ... }

// Composição (flexível)
class NotificationService {
  constructor(private sender: MessageSender) { }
}
```

### Tell, Don't Ask

Não pergunte o estado de um objeto para tomar uma decisão — diga ao objeto o que fazer. Isso respeita encapsulamento e reduz acoplamento.

```java
// Ask (ruim)
if (order.getStatus() == PENDING) { order.setStatus(APPROVED); }

// Tell (bom)
order.approve();
```

## Quando usar

- **OOP puro:** sistemas com domínio complexo (DDD), quando o modelo de negócio mapeia naturalmente para objetos
- **Interfaces:** quando precisa de polimorfismo, especialmente com Dependency Injection
- **Classes abstratas:** quando quer compartilhar implementação entre subclasses (Template Method)
- **Composição:** quase sempre preferível a herança

## Armadilhas comuns

- **Herança profunda:** hierarquias com 4+ níveis são um code smell. Prefira composição.
- **Anemic Domain Model:** classes que são só getters/setters sem comportamento. Coloque a lógica de negócio nas entidades.
- **God Class:** classe que faz tudo. Viola Single Responsibility.
- **Over-engineering:** criar abstrações para problemas que não existem (ainda). YAGNI.
- **Confundir interface com classe abstrata:** interface define contrato, classe abstrata compartilha implementação.

## Na prática (da minha experiência)

> Em Java com Spring Boot, uso Dependency Injection extensivamente — o Spring injeta implementações concretas via interface. Isso facilita testes (injetar mocks) e permite trocar implementações sem alterar o código consumidor. No MedEspecialista, a interface `NotificationSender` é implementada por `EmailSender`, `SmsSender` e `PushSender` — adicionar um novo canal é criar uma classe nova sem tocar no código existente.

## How to explain in English

"Object-Oriented Programming is the foundation of how I structure backend applications. In my Java and TypeScript work, I rely heavily on SOLID principles — especially Single Responsibility and Dependency Inversion.

For example, in Spring Boot applications, I design services around interfaces. A `PaymentProcessor` interface might have implementations for different payment providers. This makes the code easy to test — I can inject a mock implementation in tests — and easy to extend — adding a new provider means creating a new class without modifying existing code.

I'm pragmatic about OOP. I don't create abstract hierarchies for their own sake. I prefer composition over inheritance because it's more flexible. If two classes share behavior, I'd rather extract a shared service and inject it than create a class hierarchy. The goal is maintainable, testable code — not an elegant class diagram."

### Key vocabulary

- encapsulamento → encapsulation: hiding internal state
- herança → inheritance: extending a base class
- polimorfismo → polymorphism: same interface, different behavior
- abstração → abstraction: hiding implementation details
- composição → composition: building complex objects from simpler ones
- inversão de dependência → dependency inversion: depend on abstractions
- princípio da responsabilidade única → single responsibility principle
- modelo de domínio anêmico → anemic domain model

## Recursos

- [Tell Don't Ask — Martin Fowler](https://martinfowler.com/bliki/TellDontAsk.html)
- [SOLID Principles — Uncle Bob](https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html)
- [Composition over Inheritance — Wikipedia](https://en.wikipedia.org/wiki/Composition_over_inheritance)

## Veja também

- [[Design Patterns]]
- [[Arquitetura de Software]]
- [[Java Fundamentals]]
