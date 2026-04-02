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

Guia completo da linguagem Java — do básico ao moderno (Java 8 → 21+).

## O que é

Java é uma linguagem orientada a objetos, fortemente tipada, com garbage collection automático e o princípio "write once, run anywhere" via JVM. Criada em 1995 pela Sun Microsystems, hoje mantida pela Oracle. Usada massivamente em sistemas enterprise, microserviços, Android, e big data.

---

## JVM (Java Virtual Machine)

- **Compilação:** `.java` → `javac` → `.class` (bytecode) → JVM interpreta/JIT compila
- **Memory areas:**
  - **Heap:** objetos e arrays. Gerenciado pelo GC.
  - **Stack:** variáveis locais e chamadas de método. Uma stack por thread.
  - **Metaspace:** metadata de classes (substituiu PermGen no Java 8)
- **Garbage Collection:** G1 (default Java 9+), ZGC (baixa latência, Java 15+), Shenandoah
- **JIT Compiler:** bytecode → código nativo em runtime. Otimiza hot paths (C1 para startup, C2 para pico)

---

## Sintaxe básica e tipos

### Tipos primitivos

| Tipo | Tamanho | Range | Wrapper | Default |
| --- | --- | --- | --- | --- |
| `byte` | 8 bits | -128 a 127 | `Byte` | 0 |
| `short` | 16 bits | -32K a 32K | `Short` | 0 |
| `int` | 32 bits | -2B a 2B | `Integer` | 0 |
| `long` | 64 bits | ±9.2×10¹⁸ | `Long` | 0L |
| `float` | 32 bits | ±3.4×10³⁸ | `Float` | 0.0f |
| `double` | 64 bits | ±1.7×10³⁰⁸ | `Double` | 0.0 |
| `char` | 16 bits | Unicode | `Character` | '\u0000' |
| `boolean` | 1 bit | true/false | `Boolean` | false |

**Autoboxing:** conversão automática entre primitivo e wrapper. Cuidado em loops (cria objetos).

### Igualdade

- **`==`** compara referência (mesmo objeto na memória)
- **`.equals()`** compara conteúdo (valor)
- **String pool:** `"hello" == "hello"` → `true` (interned), mas `new String("hello") == "hello"` → `false`
- **Contrato hashCode/equals:** se `a.equals(b)`, então `a.hashCode() == b.hashCode()`. Quebrar isso quebra HashMap.

### Casting e promoção

```java
// Promoção automática (widening): sem perda
int x = 10;
long y = x;        // int → long OK
double z = y;      // long → double OK

// Casting explícito (narrowing): pode perder dados
double d = 9.99;
int i = (int) d;   // 9 (trunca, não arredonda)
```

### Variáveis e escopo

- **Local:** dentro de método, sem valor default, deve ser inicializada
- **Instance (campo):** dentro da classe, tem valor default (0, null, false)
- **Static (classe):** compartilhada entre instâncias, acessada via `Class.field`
- **`var` (Java 10+):** inferência de tipo local. `var list = new ArrayList<String>();`
- **`final`:** valor não pode ser reatribuído (mas objetos mutáveis ainda podem ser alterados internamente)

---

## Estruturas de controle

### Condicionais

```java
// if-else
if (age >= 18) {
    status = "adult";
} else if (age >= 13) {
    status = "teen";
} else {
    status = "child";
}

// Ternário
String status = age >= 18 ? "adult" : "minor";

// Switch expression (Java 14+)
String result = switch (day) {
    case MONDAY, FRIDAY    -> "Work hard";
    case SATURDAY, SUNDAY  -> "Rest";
    default                -> "Normal day";
};
```

### Loops

```java
// for clássico
for (int i = 0; i < 10; i++) { ... }

// for-each (enhanced for)
for (String name : names) { ... }

// while
while (condition) { ... }

// do-while (executa ao menos uma vez)
do { ... } while (condition);

// break e continue
for (int i = 0; i < 100; i++) {
    if (i == 50) break;       // sai do loop
    if (i % 2 == 0) continue; // pula para próxima iteração
}
```

---

## Strings

Strings são **imutáveis** em Java. Toda operação retorna uma nova String.

```java
String s = "Hello";
s.length();                    // 5
s.charAt(0);                   // 'H'
s.substring(0, 3);             // "Hel"
s.toLowerCase();               // "hello"
s.contains("ell");             // true
s.indexOf("lo");               // 3
s.replace("Hello", "Hi");     // "Hi"
s.split(",");                  // String[]
s.strip();                     // remove espaços (Java 11+)
s.isBlank();                   // true se vazio ou só espaços (Java 11+)
```

**StringBuilder** — para concatenação em loops (mutável, mais eficiente):

```java
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i).append(", ");
}
String result = sb.toString();
// NÃO fazer: result += i; em loop (cria 1000 Strings)
```

**Formatted strings (Java 15+):**

```java
String msg = "Patient %s, age %d".formatted(name, age);
// ou
String msg = STR."Patient \{name}, age \{age}"; // String templates (preview Java 21+)
```

---

## Arrays

Coleções de tamanho fixo. Tipo definido em compile-time.

```java
// Declaração e inicialização
int[] numbers = new int[5];          // [0, 0, 0, 0, 0]
int[] primes = {2, 3, 5, 7, 11};    // literal
String[] names = new String[3];      // [null, null, null]

// Acesso
primes[0];        // 2
primes.length;    // 5 (propriedade, não método)

// Multidimensional
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6}
};
matrix[1][2];     // 6

// Iterar
for (int n : primes) { System.out.println(n); }

// Utilitários
Arrays.sort(numbers);
Arrays.fill(numbers, 0);
Arrays.copyOf(primes, 10);          // copia e expande
Arrays.asList(primes);              // converte para List (tamanho fixo)
List.of(1, 2, 3);                   // List imutável (Java 9+)
```

---

## OOP em Java

### Classes e objetos

```java
public class Patient {
    // Campos (instance variables)
    private String name;
    private final LocalDate birthDate;
    private static int count = 0;   // compartilhado entre instâncias

    // Construtor
    public Patient(String name, LocalDate birthDate) {
        this.name = name;
        this.birthDate = birthDate;
        count++;
    }

    // Construtor sobrecarregado (overloaded)
    public Patient(String name) {
        this(name, LocalDate.now());  // chama outro construtor
    }

    // Métodos
    public String getName() { return name; }
    public int getAge() {
        return Period.between(birthDate, LocalDate.now()).getYears();
    }

    // Método estático
    public static int getCount() { return count; }
}
```

### Modificadores de acesso

| Modificador | Classe | Package | Subclass | Mundo |
| --- | --- | --- | --- | --- |
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| (default) | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

### Herança

```java
public class Doctor extends Patient {
    private String specialty;

    public Doctor(String name, LocalDate birthDate, String specialty) {
        super(name, birthDate);   // chama construtor do pai
        this.specialty = specialty;
    }

    @Override
    public String toString() {    // sobrescreve método do pai
        return "Dr. " + getName() + " (" + specialty + ")";
    }
}
```

**Regras de herança:**
- Java suporta herança simples (uma superclass). Múltipla apenas via interfaces.
- `final class` não pode ser estendida. `final method` não pode ser sobrescrito.
- Construtores não são herdados — subclass deve chamar `super()`.

### Overriding vs Overloading

| Aspecto | Overriding (sobrescrita) | Overloading (sobrecarga) |
| --- | --- | --- |
| Onde | Subclass redefine método da superclass | Mesma classe, métodos com mesmo nome |
| Assinatura | Mesma assinatura | Parâmetros diferentes (tipo ou quantidade) |
| Retorno | Mesmo tipo ou covariante (subtipo) | Pode ser diferente |
| Acesso | Igual ou mais permissivo | Qualquer |
| `@Override` | Obrigatório (por convenção) | Não se aplica |
| Binding | Runtime (dynamic dispatch) | Compile-time (static) |
| `static` | Não pode ser sobrescrito (hidden) | Pode ser sobrecarregado |

```java
// Overloading — mesmo nome, parâmetros diferentes
public double calculate(double price) { return price * 1.1; }
public double calculate(double price, double discount) { return price * (1 - discount); }

// Overriding — subclass redefine comportamento
@Override
public String toString() { return "Custom: " + name; }
```

### Classes abstratas vs Interfaces

```java
// Classe abstrata — pode ter estado e implementação
public abstract class Notification {
    protected String recipient;

    public Notification(String recipient) {
        this.recipient = recipient;
    }

    public abstract void send(String message); // subclass implementa

    public void log(String message) {          // implementação compartilhada
        System.out.println("Sent to " + recipient + ": " + message);
    }
}

// Interface — contrato puro (Java 8+: pode ter default methods)
public interface Sendable {
    void send(String message);                  // abstrato

    default void retry(String message, int times) {  // default method
        for (int i = 0; i < times; i++) {
            try { send(message); return; }
            catch (Exception e) { /* retry */ }
        }
    }

    static Sendable noOp() {                   // static method
        return message -> {};
    }
}
```

**Quando usar:**
- **Interface:** definir contrato. Uma classe pode implementar N interfaces.
- **Classe abstrata:** compartilhar estado e implementação entre subclasses. Herança simples.

### Enums

```java
public enum OrderStatus {
    PENDING("Pendente"),
    CONFIRMED("Confirmado"),
    SHIPPED("Enviado"),
    DELIVERED("Entregue"),
    CANCELLED("Cancelado");

    private final String label;

    OrderStatus(String label) { this.label = label; }

    public String getLabel() { return label; }

    // Enum pode ter métodos
    public boolean isFinal() {
        return this == DELIVERED || this == CANCELLED;
    }
}

// Uso
OrderStatus status = OrderStatus.PENDING;
OrderStatus.valueOf("PENDING");    // parse de String
OrderStatus.values();              // todos os valores
```

---

## Annotations

Metadata no código. Processadas em compile-time ou runtime.

```java
// Built-in
@Override          // verifica que está sobrescrevendo
@Deprecated        // marca como obsoleto
@SuppressWarnings  // silencia warnings
@FunctionalInterface // garante que interface tem 1 método abstrato

// Custom annotation
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface NotEmpty {
    String message() default "Field cannot be empty";
}
```

**Em Spring:** `@Service`, `@Repository`, `@RestController`, `@Autowired`, `@Transactional` — annotations são o mecanismo principal de configuração.

> **Fonte:** [O que são anotações no Java? (vídeo)](https://www.youtube.com/watch?v=d7oJwcGJWUk)

---

## Collections Framework

### Hierarquia

```text
Iterable
  └── Collection
       ├── List     → ArrayList, LinkedList, Vector
       ├── Set      → HashSet, LinkedHashSet, TreeSet
       └── Queue    → PriorityQueue, ArrayDeque, LinkedList

Map (não é Collection)
  └── HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap, Hashtable
```

### Comparativo detalhado

| Interface | Impl | Estrutura interna | Ordenado | Duplicatas | Null | Thread-safe | Quando usar |
| --- | --- | --- | --- | --- | --- | --- | --- |
| List | ArrayList | Array dinâmico | Inserção | Sim | Sim | Não | Default para listas |
| List | LinkedList | Lista duplamente ligada | Inserção | Sim | Sim | Não | Inserção/remoção no início |
| Set | HashSet | HashMap interno | Não | Não | 1 null | Não | Deduplicação |
| Set | LinkedHashSet | HashMap + lista ligada | Inserção | Não | 1 null | Não | Dedup mantendo ordem |
| Set | TreeSet | Red-Black tree | Natural/Comparator | Não | Não | Não | Conjunto ordenado |
| Map | HashMap | Array de buckets + lista/árvore | Não | Keys únicas | 1 null key | Não | Default para mapas |
| Map | LinkedHashMap | HashMap + lista ligada | Inserção ou acesso | Keys únicas | 1 null key | Não | Cache LRU |
| Map | TreeMap | Red-Black tree | Natural/Comparator | Keys únicas | Não | Não | Mapa ordenado |
| Map | ConcurrentHashMap | Segments com locks | Não | Keys únicas | Não | Sim | Concorrência |
| Queue | PriorityQueue | Binary heap | Por prioridade | Sim | Não | Não | Top-K, scheduling |
| Queue | ArrayDeque | Array circular | FIFO/LIFO | Sim | Não | Não | Stack ou Queue |

### Operações comuns

```java
// List
List<String> names = new ArrayList<>(List.of("Ana", "Bruno", "Carlos"));
names.add("Diana");
names.get(0);              // "Ana"
names.remove("Bruno");
names.contains("Ana");     // true
names.indexOf("Carlos");   // 1
names.sort(Comparator.naturalOrder());
names.subList(0, 2);       // view (não cópia!)

// Set
Set<String> unique = new HashSet<>(names);
unique.add("Ana");         // false (já existe)

// Map
Map<String, Integer> scores = new HashMap<>();
scores.put("Ana", 95);
scores.getOrDefault("Bob", 0);         // 0
scores.putIfAbsent("Ana", 100);        // não sobrescreve
scores.merge("Ana", 5, Integer::sum);  // 95 + 5 = 100
scores.computeIfAbsent("Bob", k -> 0); // cria se não existe
scores.forEach((k, v) -> System.out.println(k + ": " + v));

// Coleções imutáveis (Java 9+)
List<String> immutable = List.of("a", "b", "c");
Map<String, Integer> immutableMap = Map.of("key", 1);
Set<String> immutableSet = Set.of("x", "y");
// .add(), .put() lançam UnsupportedOperationException

// Collections utilitários
Collections.unmodifiableList(list);   // view imutável (list original ainda é mutável!)
Collections.synchronizedList(list);   // wrapper thread-safe
Collections.singletonList("only");    // lista com 1 elemento
Collections.emptyList();              // lista vazia imutável
```

### Comparable vs Comparator

```java
// Comparable — ordem natural, implementado NA classe
public class Patient implements Comparable<Patient> {
    @Override
    public int compareTo(Patient other) {
        return this.name.compareTo(other.name);
    }
}

// Comparator — ordem customizada, FORA da classe
patients.sort(Comparator.comparing(Patient::getName));
patients.sort(Comparator.comparing(Patient::getAge).reversed());
patients.sort(Comparator.comparing(Patient::getSpecialty)
                        .thenComparing(Patient::getName));
```

### Iteração

```java
// for-each (mais comum)
for (Patient p : patients) { ... }

// Iterator (permite remover durante iteração)
Iterator<Patient> it = patients.iterator();
while (it.hasNext()) {
    Patient p = it.next();
    if (p.isInactive()) it.remove();
}

// forEach com lambda
patients.forEach(p -> System.out.println(p.getName()));

// Stream (processamento funcional)
patients.stream().filter(Patient::isActive).toList();
```

---

## Lambdas e Interfaces Funcionais

Introduzidas no Java 8. Lambdas são funções anônimas que implementam interfaces funcionais.

### Sintaxe

```java
// Sem lambda (classe anônima)
Comparator<String> comp = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
};

// Com lambda
Comparator<String> comp = (a, b) -> a.length() - b.length();

// Method reference
Comparator<String> comp = Comparator.comparingInt(String::length);
```

### Interfaces funcionais do `java.util.function`

| Interface | Assinatura | Uso |
| --- | --- | --- |
| `Function<T,R>` | `R apply(T t)` | Transformar T em R |
| `Predicate<T>` | `boolean test(T t)` | Filtrar/testar condição |
| `Consumer<T>` | `void accept(T t)` | Ação sem retorno (forEach) |
| `Supplier<T>` | `T get()` | Factory, lazy evaluation |
| `UnaryOperator<T>` | `T apply(T t)` | Transformar mantendo tipo |
| `BiFunction<T,U,R>` | `R apply(T t, U u)` | Transformar dois argumentos |
| `BiPredicate<T,U>` | `boolean test(T t, U u)` | Testar com dois argumentos |

### Method references

```java
// Static method
Function<String, Integer> parse = Integer::parseInt;

// Instance method de um tipo
Function<String, String> upper = String::toUpperCase;

// Instance method de um objeto
Consumer<String> printer = System.out::println;

// Constructor
Supplier<ArrayList<String>> factory = ArrayList::new;
```

### Composição de funções

```java
Function<String, String> trim = String::strip;
Function<String, String> lower = String::toLowerCase;
Function<String, String> process = trim.andThen(lower);

Predicate<Patient> active = Patient::isActive;
Predicate<Patient> senior = p -> p.getAge() > 60;
Predicate<Patient> activeSenior = active.and(senior);
```

---

## Streams API

Pipeline funcional para processar coleções. Introduzida no Java 8.

### Anatomia de um Stream

```text
Source (List, Array, File, Generator)
  → Operações intermediárias (lazy, retornam Stream)
  → Operação terminal (eager, produz resultado)
```

### Operações intermediárias

```java
stream.filter(p -> p.isActive())         // filtrar
      .map(Patient::getName)              // transformar
      .flatMap(name -> name.chars().boxed()) // 1→N (achatar)
      .distinct()                         // remover duplicatas
      .sorted()                           // ordenar (natural)
      .sorted(Comparator.comparing(Patient::getAge)) // ordenar custom
      .peek(System.out::println)          // debug (não usar em prod)
      .limit(10)                          // primeiros N
      .skip(5)                            // pular N
      .takeWhile(p -> p.getAge() < 60)    // até condição falhar (Java 9+)
      .dropWhile(p -> p.getAge() < 18)    // descartar até condição falhar
```

### Operações terminais

```java
// Coletar
.toList()                                 // Java 16+ (imutável)
.collect(Collectors.toList())             // mutável
.collect(Collectors.toSet())
.collect(Collectors.toMap(Patient::getId, Function.identity()))
.collect(Collectors.joining(", "))        // "Ana, Bruno, Carlos"

// Agrupar
.collect(Collectors.groupingBy(Patient::getSpecialty))  // Map<String, List<Patient>>
.collect(Collectors.groupingBy(Patient::getSpecialty, Collectors.counting())) // Map<String, Long>
.collect(Collectors.partitioningBy(Patient::isActive))  // Map<Boolean, List<Patient>>

// Reduzir
.count()
.min(Comparator.comparing(Patient::getAge))  // Optional<Patient>
.max(Comparator.comparing(Patient::getAge))
.reduce(0, (sum, p) -> sum + p.getAge(), Integer::sum)

// Buscar
.findFirst()     // Optional<T> — primeiro elemento
.findAny()       // Optional<T> — qualquer (útil em parallel)
.anyMatch(p -> p.getAge() > 60)   // boolean
.allMatch(Patient::isActive)
.noneMatch(p -> p.getName().isBlank())

// Iterar
.forEach(System.out::println)     // ação sem retorno
.forEachOrdered(...)              // garante ordem em parallel
```

### Streams primitivos

Evitam boxing/unboxing. Ganho significativo de performance em grandes volumes.

```java
IntStream.range(0, 100)           // 0 a 99
IntStream.rangeClosed(1, 100)     // 1 a 100
IntStream.of(1, 2, 3)

patients.stream()
    .mapToInt(Patient::getAge)    // IntStream
    .average()                    // OptionalDouble
    .orElse(0.0);

// Converter de volta
IntStream.range(0, 10).boxed()    // Stream<Integer>
```

### Exemplo completo

```java
// Relatório: top 5 especialidades com mais pacientes ativos
Map<String, Long> topSpecialties = patients.stream()
    .filter(Patient::isActive)
    .collect(Collectors.groupingBy(
        Patient::getSpecialty,
        Collectors.counting()
    ))
    .entrySet().stream()
    .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
    .limit(5)
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (a, b) -> a,
        LinkedHashMap::new  // mantém ordem
    ));
```

### Checklist de performance

1. `collect()` guarda tudo na memória? Pode reduzir em passo único (`sum`, `count`, `max`)?
2. Usar Streams primitivos (`IntStream`, `LongStream`) para evitar boxing
3. `sorted()` ou `distinct()` em dados grandes carrega tudo em memória
4. Filtros mais seletivos no início da pipeline
5. Operações de curto-circuito (`findFirst`, `limit`, `anyMatch`) quando possível
6. `parallelStream()` — só quando o overhead de threading compensa (dados grandes, operação CPU-intensive)
7. `flatMap` pode expandir demais o volume — monitorar

> **Fontes:**
> - [Java Streams — exemplo prático](https://computaria.gitlab.io/blog/2025/04/27/java-streams-exemplo)
> - [Java moderno com Peano](https://computaria.gitlab.io/blog/2025/03/10/peano-java-moderno)

---

## Date/Time API (Java 8+)

Substituiu o problemático `Date`/`Calendar`. Imutável, thread-safe.

```java
// Data
LocalDate today = LocalDate.now();
LocalDate birth = LocalDate.of(1985, 3, 15);
LocalDate parsed = LocalDate.parse("2026-04-01");

// Hora
LocalTime now = LocalTime.now();
LocalTime appointment = LocalTime.of(14, 30);

// Data + Hora
LocalDateTime dateTime = LocalDateTime.of(today, appointment);

// Com timezone
ZonedDateTime zonedNow = ZonedDateTime.now(ZoneId.of("America/Sao_Paulo"));
Instant instant = Instant.now();  // timestamp UTC (para persistência)

// Duração e período
Duration duration = Duration.between(start, end);   // horas, minutos, segundos
Period period = Period.between(birth, today);         // anos, meses, dias
int age = period.getYears();

// Formatação
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
String formatted = dateTime.format(fmt);
LocalDateTime parsed2 = LocalDateTime.parse("01/04/2026 14:30", fmt);

// Manipulação (imutável — retorna nova instância)
LocalDate nextWeek = today.plusWeeks(1);
LocalDate lastMonth = today.minusMonths(1);
LocalDate firstDayOfMonth = today.withDayOfMonth(1);
```

**Regra prática:**
- `LocalDate` / `LocalTime` / `LocalDateTime` — quando timezone não importa
- `ZonedDateTime` — quando precisa de timezone (agendamentos internacionais)
- `Instant` — para persistência e cálculos de duração (sempre UTC)

---

## I/O (Arquivos)

### java.nio.file (moderno — preferir)

```java
// Ler arquivo inteiro
String content = Files.readString(Path.of("data.txt"));
List<String> lines = Files.readAllLines(Path.of("data.txt"));

// Ler arquivo grande (streaming — não carrega tudo na memória)
try (Stream<String> stream = Files.lines(Path.of("large.csv"))) {
    long count = stream.filter(line -> line.contains("ERROR")).count();
}

// Escrever
Files.writeString(Path.of("output.txt"), "Hello");
Files.write(Path.of("output.txt"), lines);

// Operações de diretório
Files.exists(path);
Files.createDirectories(Path.of("a/b/c"));
Files.list(Path.of("."))             // Stream<Path> (nível 1)
     .filter(Files::isRegularFile)
     .forEach(System.out::println);
Files.walk(Path.of("."))             // Stream<Path> (recursivo)
     .filter(p -> p.toString().endsWith(".java"))
     .forEach(System.out::println);

// Copiar e mover
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(source, target);
Files.delete(path);
```

### Try-with-resources

```java
try (var reader = new BufferedReader(new FileReader("data.csv"));
     var writer = new BufferedWriter(new FileWriter("output.csv"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        writer.write(processLine(line));
        writer.newLine();
    }
} // ambos fechados automaticamente
```

---

## Exceções

Hierarquia: `Throwable` → `Error` (sistema, não tratar) e `Exception` (aplicação).

```text
Throwable
  ├── Error (OutOfMemoryError, StackOverflowError) — não capturar
  └── Exception
       ├── IOException, SQLException — checked (deve tratar)
       └── RuntimeException
            ├── NullPointerException
            ├── IllegalArgumentException
            ├── IllegalStateException
            └── IndexOutOfBoundsException — unchecked
```

**Checked vs Unchecked:**
- **Checked:** o compilador obriga a tratar (try-catch ou throws). Para erros recuperáveis de I/O.
- **Unchecked (RuntimeException):** erros de programação. Não precisa declarar.

**Custom exceptions:**

```java
public class PatientNotFoundException extends RuntimeException {
    public PatientNotFoundException(Long id) {
        super("Patient not found: " + id);
    }
}
```

**Boas práticas:**
- Capturar exceções específicas, nunca `catch (Exception e)` genérico
- Não silenciar exceções (catch vazio)
- Usar try-with-resources para qualquer `AutoCloseable`
- Em APIs REST, traduzir exceções para HTTP status codes via `@ExceptionHandler`
- Preservar a causa original: `throw new ServiceException("msg", originalException)`

> **Fontes:**
> - [Java Exceptions — Baeldung](https://www.baeldung.com/java-exceptions)
> - [Checked vs Unchecked — Baeldung](https://www.baeldung.com/java-checked-unchecked-exceptions)
> - [Try-with-resources — Baeldung](https://www.baeldung.com/java-try-with-resources)
> - [Pensando nas Exceptions do Java](https://insights.itexto.com.br/pensando-nas-exceptions-do-java-ou-por-que-elas-sao-assim/)

---

## Optional

Wrapper para valores que podem ou não existir. Substitui `null`.

```java
Optional<Patient> patient = repository.findById(id);

String name = patient.map(Patient::getName).orElse("Desconhecido");
Patient p = patient.orElseThrow(() -> new PatientNotFoundException(id));
patient.ifPresent(pat -> sendWelcomeEmail(pat));
```

**Regras:**
- Usar como retorno de métodos que podem não ter resultado
- **Nunca** como parâmetro de método ou campo de classe
- **Nunca** usar `Optional.get()` sem verificar
- `Optional.empty()` em vez de retornar `null`

> **Fonte:** [Java Optional](https://computaria.gitlab.io/blog/2025/04/25/java-optional)

---

## Generics

Tipos parametrizados. Segurança de tipos em compile-time.

```java
// Classe genérica
public class Result<T> {
    private final T data;
    private final String error;

    public static <T> Result<T> ok(T data) { return new Result<>(data, null); }
    public static <T> Result<T> fail(String error) { return new Result<>(null, error); }
}

// Bounded types
public <T extends Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}

// Wildcards
List<? extends Number> numbers;  // leitura (producer) — pode ser List<Integer>, List<Double>
List<? super Integer> sink;      // escrita (consumer) — pode ser List<Integer>, List<Number>
```

**PECS:** Producer Extends, Consumer Super.

**Type erasure:** generics são removidos em runtime. `List<String>` e `List<Integer>` são o mesmo `List` em bytecode.

---

## Concorrência

```java
// Thread básica
Thread thread = new Thread(() -> System.out.println("Running"));
thread.start();

// ExecutorService — pool gerenciado
ExecutorService executor = Executors.newFixedThreadPool(4);
Future<String> future = executor.submit(() -> fetchData());
String result = future.get(); // bloqueia até completar

// CompletableFuture — composição assíncrona (Java 8+)
CompletableFuture.supplyAsync(() -> fetchUser(id))
    .thenApply(user -> enrichWithOrders(user))
    .thenAccept(user -> sendNotification(user))
    .exceptionally(ex -> { log.error("Failed", ex); return null; });

// Executar em paralelo
CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> fetchUser(id));
CompletableFuture<List<Order>> ordersFuture = CompletableFuture.supplyAsync(() -> fetchOrders(id));
CompletableFuture.allOf(userFuture, ordersFuture).join();
```

**Synchronized vs Lock:**
- `synchronized` — implícito, simples, automático. Suficiente para a maioria dos casos.
- `ReentrantLock` — explícito, tryLock com timeout, interruptible. Para cenários avançados.

**Volatile:** garante visibilidade entre threads, mas não atomicidade. Para flags simples.

**Virtual Threads (Java 21):** threads leves gerenciadas pela JVM. Ideais para I/O-bound.

```java
// Virtual threads — milhões de threads sem overhead
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 100_000).forEach(i ->
        executor.submit(() -> fetchData(i))
    );
}
```

---

## Features modernas por versão

### Java 8 (2014) — A revolução

- **Lambdas e interfaces funcionais** — `(x) -> x * 2`
- **Streams API** — processamento funcional de coleções
- **Optional** — alternativa a null
- **Date/Time API** — `LocalDate`, `Instant`, `Duration`
- **Default methods em interfaces** — evolução sem quebrar
- **Method references** — `String::toUpperCase`

### Java 9-10 (2017-2018)

- **Modules (JPMS)** — modularização da JVM
- **`List.of()`, `Set.of()`, `Map.of()`** — coleções imutáveis
- **`var` (Java 10)** — inferência de tipo local
- **`Optional.ifPresentOrElse()`, `Optional.or()`**
- **`Stream.ofNullable()`, `Stream.takeWhile()`, `Stream.dropWhile()`**

### Java 11-16 (2018-2021)

- **`String.isBlank()`, `.strip()`, `.lines()`, `.repeat()`** (Java 11)
- **`Files.readString()`, `Files.writeString()`** (Java 11)
- **Switch expressions** (Java 14) — `switch` retorna valor
- **Text blocks** (Java 15) — `"""multiline"""`
- **Records** (Java 16) — `record Point(int x, int y) {}`
- **Pattern matching for instanceof** (Java 16) — `if (obj instanceof String s)`
- **`Stream.toList()`** (Java 16) — substitui `collect(Collectors.toList())`

### Java 17 (2021, LTS)

- **Sealed classes** — `sealed class Shape permits Circle, Rectangle`
- **Pattern matching em switch** (preview)
- **Remoção do Applet API e Security Manager**

### Java 21 (2023, LTS)

- **Virtual Threads** — threads leves para I/O-bound
- **Record patterns** — `if (obj instanceof Point(int x, int y))`
- **Pattern matching em switch** (final)
- **Sequenced Collections** — `SequencedCollection`, `SequencedMap` com `getFirst()`, `getLast()`
- **String templates** (preview) — `STR."Hello \{name}"`

### Java 22-25+ (2024-2026)

- **Unnamed variables** — `_` para variáveis não usadas (Java 22)
- **Statements before super()** (Java 22) — validar antes de chamar construtor pai
- **Structured Concurrency** (preview) — gerenciar grupo de tarefas como unidade
- **Scoped Values** (preview) — alternativa thread-safe a ThreadLocal
- **Stream Gatherers** (Java 22) — operações intermediárias customizadas

> **Fontes:**
> - [Java features desde JDK 8 ao 21](https://advancedweb.hu/a-categorized-list-of-all-java-and-jvm-features-since-jdk-8-to-21/)
> - [Java Language Updates](https://docs.oracle.com/en/java/javase/21/language/index.html)
> - [Java Evolved](https://javaevolved.github.io/)

---

## Armadilhas comuns

- **NullPointerException:** usar `Optional` para retornos, `Objects.requireNonNull()` para validação
- **Mutabilidade acidental:** `List.of()` é imutável, `Collections.unmodifiableList()` é view (original mutável!)
- **ConcurrentModificationException:** modificar coleção durante iteração. Usar `Iterator.remove()` ou Streams.
- **Autoboxing em loops:** `Integer` em vez de `int` em loops grandes = milhões de objetos
- **String concatenação em loop:** usar `StringBuilder`, não `+=`
- **`==` para comparar Strings:** usar `.equals()`. `==` funciona com literais por causa do pool, mas falha com `new String()`
- **Parallel streams sem pensar:** ForkJoinPool compartilhado, poucos elementos = overhead > ganho

## Na prática (da minha experiência)

> Com 20+ anos em Java, a linguagem evoluiu enormemente. No MedEspecialista, uso Java 21 com Records para DTOs (elimina boilerplate), Streams para transformações de dados, e CompletableFuture para chamadas paralelas a serviços externos. A migração para Virtual Threads reduziu a necessidade de tuning de thread pools em endpoints I/O-bound. No dia a dia, Collections e Streams são 80% do que uso — o outro 20% é concorrência e I/O.

## How to explain in English

"Java has been my primary language for over 20 years, and I've seen it evolve significantly. Modern Java — 17 and beyond — is much more concise than the Java of 10 years ago. Records eliminate boilerplate for data classes, pattern matching simplifies type checking, and text blocks make working with multi-line strings natural.

The Collections Framework is something I use constantly. For most cases, ArrayList and HashMap cover 90% of needs. I reach for ConcurrentHashMap in multi-threaded scenarios and TreeMap when I need sorted iteration. Understanding the performance characteristics helps me make the right choice.

Streams and lambdas transformed how I write Java. Instead of imperative loops with mutable accumulators, I use declarative pipelines that are easier to read and parallelize. The key is knowing when Streams add clarity versus when a simple for-loop is better.

For concurrency, Virtual Threads in Java 21 are a game-changer. In traditional Java, each thread maps to an OS thread, limiting you to tens of thousands of concurrent connections. Virtual Threads are managed by the JVM and are much cheaper — you can have millions. This simplifies I/O-bound microservices enormously."

### Key vocabulary

- máquina virtual → JVM (Java Virtual Machine)
- coleta de lixo → garbage collection (GC)
- tipo genérico → generic type
- thread virtual → virtual thread (Project Loom)
- registro → record: classe imutável para dados
- fluxo → stream: pipeline de processamento funcional
- classe selada → sealed class
- inferência de tipo → type inference (`var`)
- sobrescrita → overriding: redefinir método na subclass
- sobrecarga → overloading: mesmo nome, parâmetros diferentes
- interface funcional → functional interface: interface com 1 método abstrato
- referência a método → method reference: `Class::method`

## Recursos

- [Java Language Updates](https://docs.oracle.com/en/java/javase/21/language/index.html) — features por versão
- [JDK 22 Documentation](https://docs.oracle.com/en/java/javase/22/)
- [Java features desde JDK 8 ao 21](https://advancedweb.hu/a-categorized-list-of-all-java-and-jvm-features-since-jdk-8-to-21/)
- [Java Evolved](https://javaevolved.github.io/) — evolução da linguagem
- [O que são anotações no Java? (vídeo)](https://www.youtube.com/watch?v=d7oJwcGJWUk)
- [[Trilha Java]] — trilha de aprendizado completa
- [[What should you do to stand out as a Java-Spring Boot Developer]]

## Veja também

- [[Spring Boot]]
- [[Testes em Java]]
- [[Orientação a Objetos]]
- [[Design Patterns]]
- [[Kafka]]
- [[JavaFX]]
