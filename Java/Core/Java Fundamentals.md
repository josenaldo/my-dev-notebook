---
title: "Java Fundamentals"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - java
  - entrevista
publish: false
---

# Java Fundamentals

Os conceitos core de Java que todo senior precisa dominar — da JVM ao Collections Framework.

## O que é

Java é uma linguagem orientada a objetos, fortemente tipada, com garbage collection automático e o princípio "write once, run anywhere" via JVM. Para entrevistas, o foco é em: JVM internals, Collections, concorrência, Streams, records, e features modernas (Java 17+).

## Como funciona

### JVM (Java Virtual Machine)

- **Compilação:** `.java` → `javac` → `.class` (bytecode) → JVM interpreta/JIT compila
- **Memory areas:** Heap (objetos), Stack (variáveis locais, chamadas de método), Metaspace (metadata de classes)
- **Garbage Collection:** algoritmos como G1 (default Java 9+), ZGC (baixa latência), Shenandoah
- **JIT Compiler:** bytecode → código nativo em runtime. Otimiza hot paths.

### Tipos e Igualdade

- **Primitivos:** `int`, `long`, `double`, `boolean`, `char` — stack, valor direto
- **Wrapper classes:** `Integer`, `Long` — heap, boxing/unboxing automático
- **`==` vs `.equals()`:** `==` compara referência (mesmo objeto), `.equals()` compara conteúdo
- **String pool:** `"hello" == "hello"` é `true` (interned), mas `new String("hello") == "hello"` é `false`
- **Contrato hashCode/equals:** se `a.equals(b)`, então `a.hashCode() == b.hashCode()`. Essencial para HashMap.

### Collections Framework

| Interface | Implementação | Ordenado | Permite null | Thread-safe |
| --- | --- | --- | --- | --- |
| List | ArrayList | Por inserção | Sim | Não |
| List | LinkedList | Por inserção | Sim | Não |
| Set | HashSet | Não | 1 null | Não |
| Set | TreeSet | Natural/Comparator | Não | Não |
| Map | HashMap | Não | 1 null key | Não |
| Map | TreeMap | Natural/Comparator | Não (key) | Não |
| Map | ConcurrentHashMap | Não | Não | Sim |
| Queue | PriorityQueue | Por prioridade | Não | Não |
| Queue | ArrayDeque | FIFO/LIFO | Não | Não |

**Regra prática:** `ArrayList` para listas, `HashMap` para associação, `ConcurrentHashMap` para concorrência.

### Streams API

Pipeline funcional para processar coleções de forma declarativa.

```java
var topDoctors = doctors.stream()
    .filter(d -> d.getSpecialty().equals("Cardiology"))
    .sorted(Comparator.comparing(Doctor::getRating).reversed())
    .limit(10)
    .map(Doctor::getName)
    .toList(); // Java 16+
```

- **Lazy evaluation:** operações intermediárias só executam quando o terminal é chamado
- **Parallel streams:** `parallelStream()` — cuidado com shared state e overhead de thread pool
- **Collectors:** `toList()`, `toMap()`, `groupingBy()`, `joining()`, `partitioningBy()`

### Concorrência

- **Thread:** unidade de execução. `Thread`, `Runnable`, `Callable`
- **ExecutorService:** pool de threads gerenciado. `Executors.newFixedThreadPool(n)`
- **CompletableFuture:** composição assíncrona. `.thenApply()`, `.thenCompose()`, `.exceptionally()`
- **Synchronized vs Lock:** `synchronized` (implícito) vs `ReentrantLock` (explícito, mais controle)
- **Volatile:** garante visibilidade entre threads, mas não atomicidade
- **Virtual Threads (Java 21):** threads leves gerenciadas pela JVM, não pelo OS. Ideal para I/O-bound.

### Features modernas (Java 17+)

- **Records:** classes imutáveis para dados. `record Patient(String name, String email) {}`
- **Sealed classes:** restringe quais classes podem herdar. Útil com pattern matching.
- **Pattern matching:** `if (obj instanceof String s)` — cast automático
- **Text blocks:** `""" multiline string """`
- **Switch expressions:** retorna valor, suporta pattern matching (Java 21)

### Generics

- **Bounded types:** `<T extends Comparable<T>>`
- **Wildcards:** `<? extends Number>` (upper bound), `<? super Integer>` (lower bound)
- **Type erasure:** generics são removidos em compile-time. Não existe `List<String>.class` em runtime.
- **PECS:** Producer Extends, Consumer Super

## Quando usar

- **Java:** sistemas enterprise, microserviços com Spring Boot, aplicações de alta concorrência
- **ArrayList vs LinkedList:** quase sempre ArrayList (melhor cache locality)
- **HashMap vs TreeMap:** HashMap para lookup, TreeMap quando precisa de ordenação
- **Streams vs for loop:** Streams para pipelines complexos, for loop para operações simples com side effects
- **Virtual Threads:** I/O-bound tasks (HTTP calls, DB queries). Não para CPU-bound.

## Armadilhas comuns

- **NullPointerException:** usar `Optional` para retornos que podem ser nulos. Nunca `Optional` como parâmetro.
- **Mutabilidade acidental:** `Collections.unmodifiableList()` retorna view, não cópia. `List.of()` é realmente imutável.
- **ConcurrentModificationException:** modificar coleção durante iteração. Usar `Iterator.remove()` ou Streams.
- **Parallel streams sem pensar:** o ForkJoinPool compartilhado pode causar problemas. Poucos elementos = overhead maior que o ganho.
- **Autoboxing em loops:** `Integer` em vez de `int` em loops grandes = milhões de objetos desnecessários.

## Na prática (da minha experiência)

> Com 20+ anos em Java, a linguagem evoluiu enormemente. No MedEspecialista, uso Java 21 com Records para DTOs (elimina boilerplate), Streams para transformações de dados, e CompletableFuture para chamadas paralelas a serviços externos. A migração para Virtual Threads reduziu a necessidade de tuning de thread pools em endpoints I/O-bound, simplificando a configuração de produção.

## How to explain in English

"Java has been my primary language for over 20 years, and I've seen it evolve significantly. Modern Java — 17 and beyond — is much more concise than the Java of 10 years ago. Records eliminate boilerplate for data classes, pattern matching simplifies type checking, and text blocks make working with multi-line strings natural.

The Collections Framework is something I use constantly. For most cases, ArrayList and HashMap cover 90% of needs. I reach for ConcurrentHashMap in multi-threaded scenarios and TreeMap when I need sorted iteration. Understanding the performance characteristics — ArrayList's O(1) random access versus LinkedList's O(1) insertion — helps me make the right choice.

For concurrency, I'm excited about Virtual Threads in Java 21. In traditional Java, each thread maps to an OS thread, which limits you to tens of thousands of concurrent connections. Virtual Threads are managed by the JVM and are much cheaper — you can have millions. This is a game-changer for I/O-bound microservices where most threads are waiting on network calls."

### Key vocabulary

- máquina virtual → JVM (Java Virtual Machine)
- coleta de lixo → garbage collection (GC)
- tipo genérico → generic type
- thread virtual → virtual thread (Project Loom)
- registro → record: classe imutável para dados
- fluxo → stream: pipeline de processamento funcional
- classe selada → sealed class
- inferência de tipo → type inference (`var`)

## Recursos

- [Java Language Updates](https://docs.oracle.com/en/java/javase/21/language/index.html) — features por versão
- [[Trilha Java]] — trilha de aprendizado completa
- [[What should you do to stand out as a Java-Spring Boot Developer]]

## Veja também

- [[Spring Boot]]
- [[Testes em Java]]
- [[Orientação a Objetos]]
- [[Kafka]]
