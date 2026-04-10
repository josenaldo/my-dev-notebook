---
title: "Spring Boot"
created: 2026-04-01
updated: 2026-04-10
type: concept
status: seedling
tags:
  - java
  - backend
  - entrevista
publish: false
---

# Spring Boot

O framework Java mais usado para microserviços e aplicações web — autoconfiguração, dependency injection, e ecossistema maduro.

## O que é

Spring Boot é uma extensão do Spring Framework que simplifica a configuração e o deployment de aplicações Java. Ele fornece autoconfiguração, servidor embarcado (Tomcat/Netty), e um ecossistema de módulos (Spring Data, Spring Security, Spring Cloud) que cobrem praticamente todas as necessidades de um backend moderno.

## Como funciona

### Conceitos core

**Dependency Injection (DI):** o Spring IoC Container gerencia o ciclo de vida dos objetos (beans) e injeta dependências automaticamente.

```java
@Service
public class AppointmentService {
    private final PatientRepository patientRepo;
    private final NotificationSender notifier;

    // Constructor injection (recomendado)
    public AppointmentService(PatientRepository patientRepo,
                              NotificationSender notifier) {
        this.patientRepo = patientRepo;
        this.notifier = notifier;
    }
}
```

**Bean Scopes:**

| Scope | Descrição |
| --- | --- |
| singleton | Uma instância por contexto (default) |
| prototype | Nova instância a cada injeção |
| request | Uma por HTTP request (web) |
| session | Uma por HTTP session (web) |

**Autoconfiguração:** Spring Boot detecta dependências no classpath e configura automaticamente. `spring-boot-starter-data-jpa` no pom.xml → DataSource, EntityManager, TransactionManager configurados.

### Camadas típicas

```text
Controller (@RestController)
    ↓ DTO
Service (@Service)
    ↓ Entity
Repository (@Repository / Spring Data JPA)
    ↓ SQL
Database
```

### Spring Data JPA

```java
public interface PatientRepository extends JpaRepository<Patient, Long> {
    // Query derivada do nome do método
    List<Patient> findBySpecialtyAndActive(String specialty, boolean active);

    // JPQL custom
    @Query("SELECT p FROM Patient p WHERE p.rating > :min ORDER BY p.rating DESC")
    List<Patient> findTopRated(@Param("min") double min);
}
```

- **Derived queries:** o Spring gera SQL a partir do nome do método
- **Pagination:** `Pageable` como parâmetro → retorna `Page<T>`
- **Specifications:** queries dinâmicas com Criteria API
- **Projections:** interfaces/DTOs para retornar apenas campos necessários

### Spring Security

- **Authentication:** quem é o usuário (JWT, OAuth2, Basic Auth)
- **Authorization:** o que o usuário pode fazer (`@PreAuthorize`, roles)
- **Security Filter Chain:** pipeline de filtros que interceptam requisições

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/public/**").permitAll()
            .requestMatchers("/api/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))
        .build();
}
```

### Profiles e Configuration

- **Profiles:** `application-dev.yml`, `application-prod.yml` — configurações por ambiente
- **`@ConfigurationProperties`:** bind de propriedades para classes Java tipadas
- **Externalized config:** variáveis de ambiente, Vault, ConfigServer

> **Fontes:**
> - [Spring Profiles — Baeldung](https://www.baeldung.com/spring-profiles)
> - [Environment Variables in Properties — Baeldung](https://www.baeldung.com/spring-boot-properties-env-variables)
> - [@ConfigurationProperties — Baeldung](https://www.baeldung.com/configuration-properties-in-spring-boot)
> - [Common Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html)

### Bean Validation

Validação declarativa com anotações Jakarta Validation:

```java
public record CreatePatientRequest(
    @NotBlank String name,
    @Email @NotBlank String email,
    @Past LocalDate birthDate,
    @Size(min = 11, max = 11) String cpf
) {}

@RestController
public class PatientController {
    @PostMapping("/patients")
    public ResponseEntity<?> create(@Valid @RequestBody CreatePatientRequest req) {
        // Se validação falhar, Spring retorna 400 automaticamente
        return ResponseEntity.created(uri).body(service.create(req));
    }
}
```

**Anotações principais:** `@NotNull`, `@NotBlank`, `@NotEmpty`, `@Size`, `@Email`, `@Min`, `@Max`, `@Past`, `@Future`, `@Pattern`

**`@Valid` vs `@Validated`:** `@Valid` (Jakarta, cascata em objetos aninhados) vs `@Validated` (Spring, suporta validation groups)

**Validador customizado:**

```java
@Target(ElementType.FIELD)
@Constraint(validatedBy = CpfValidator.class)
public @interface ValidCpf {
    String message() default "CPF inválido";
}
```

> **Fontes:**
> - [Validation in Spring Boot — Baeldung](https://www.baeldung.com/spring-boot-bean-validation)
> - [Bean Validation Basics — Baeldung](https://www.baeldung.com/java-validation)
> - [@Valid vs @Validated — Baeldung](https://www.baeldung.com/spring-valid-vs-validated)
> - [Custom Validator — Baeldung](https://www.baeldung.com/spring-mvc-custom-validator)
> - [Service Layer Validation — Baeldung](https://www.baeldung.com/spring-service-layer-validation)

### Persistência (JPA, Hibernate, Flyway)

**Spring Data JPA** abstrai JPA/Hibernate. Repositórios geram queries a partir do nome do método:

```java
public interface PatientRepository extends JpaRepository<Patient, Long> {
    List<Patient> findBySpecialtyAndActive(String specialty, boolean active);

    @Query("SELECT p FROM Patient p WHERE p.rating > :min")
    List<Patient> findTopRated(@Param("min") double min);
}
```

**HikariCP** — connection pool padrão do Spring Boot. Configurar pool size é crítico:
- Regra: `connections = (core_count * 2) + spindle_count`
- Para SSD: geralmente 10-20 conexões bastam

**Flyway** — migrações de schema versionadas:

```text
resources/db/migration/
  V1__create_patients.sql
  V2__add_email_column.sql
  V3__create_appointments.sql
```

Cada migração roda uma vez, é rastreada em tabela `flyway_schema_history`. Nunca editar migrações já executadas — criar nova.

**Lombok + JPA — cuidados:**
- `@Data` em entidades JPA gera `equals`/`hashCode` com todos os campos — perigoso com lazy loading
- Usar `@Getter @Setter @NoArgsConstructor` separados
- `@EqualsAndHashCode` deve usar apenas o ID da entidade
- `@ToString` pode disparar lazy loading — excluir relações

> **Fontes:**
> - [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
> - [findById Anti-Pattern](https://vladmihalcea.com/spring-data-jpa-findbyid/)
> - [HikariCP — About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)
> - [Flyway](https://github.com/flyway/flyway) — migrations
> - [Lombok e JPA — O que pode dar errado?](https://dev.to/eronalves1996/traducao-lombok-e-jpa-o-que-pode-dar-errado-1c6)
> - [Hibernate Natural IDs — Baeldung](https://www.baeldung.com/spring-boot-hibernate-natural-ids)

### Ferramentas do ecossistema

**MapStruct** — mapeamento entre objetos (Entity ↔ DTO) em compile-time:

```java
@Mapper(componentModel = "spring")
public interface PatientMapper {
    PatientDTO toDto(Patient entity);
    Patient toEntity(CreatePatientRequest request);
}
```

Gera implementação automaticamente. Sem reflection = mais rápido que ModelMapper.

**Spring Cloud OpenFeign** — HTTP client declarativo para comunicação entre microserviços:

```java
@FeignClient(name = "notification-service", url = "${notification.url}")
public interface NotificationClient {
    @PostMapping("/notifications")
    void send(@RequestBody NotificationRequest request);
}
```

> **Fontes:**
> - [MapStruct](https://mapstruct.org/) — documentação oficial
> - [Spring Cloud OpenFeign](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)
> - [Feign ErrorDecoder — Baeldung](https://www.baeldung.com/feign-retrieve-original-message)

### Observabilidade

- **Actuator:** endpoints `/health`, `/metrics`, `/info` para monitoramento
- **Micrometer:** métricas exportadas para Prometheus, Grafana, Datadog
- **OpenTelemetry:** tracing distribuído entre microserviços

### Testing em Spring Boot

```java
@SpringBootTest
@Testcontainers
class AppointmentServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @Autowired
    private AppointmentService service;

    @Test
    void shouldCreateAppointment() {
        var result = service.create(new AppointmentRequest(...));
        assertThat(result.getId()).isNotNull();
    }
}
```

- **`@SpringBootTest`:** carrega o contexto completo
- **`@WebMvcTest`:** só controller layer (com MockMvc)
- **`@DataJpaTest`:** só repository layer
- **Testcontainers:** bancos reais em Docker

## Troubleshooting em produção

Problemas recorrentes em aplicações Spring Boot — o tipo de pergunta que aparece em entrevistas como "you've seen this in production, how did you debug it?"

### Connection pool exausto (HikariCP)

**Sintoma:** requests travam, logs mostram `HikariPool - Connection is not available, request timed out after 30000ms`.

**Causas comuns:**
- Pool pequeno demais para a carga
- Queries lentas segurando conexões
- Transação não fechada (leak)
- `@Transactional` em método que faz chamada HTTP externa (segura conexão enquanto espera response)

**Diagnóstico:**

```java
// application.yml — habilitar métricas do HikariCP
spring:
  datasource:
    hikari:
      maximum-pool-size: 10        # default é 10
      minimum-idle: 5
      connection-timeout: 30000     # ms para esperar conexão do pool
      leak-detection-threshold: 60000  # alerta se conexão não devolvida em 60s
      metrics-tracker-factory: io.micrometer.core.instrument.binder.db.HikariCPMetricsTrackerFactory
```

**Métricas para monitorar (Micrometer/Prometheus):**
- `hikaricp_connections_active` — quantas em uso
- `hikaricp_connections_pending` — quantas esperando (se > 0 por muito tempo, pool exausto)
- `hikaricp_connections_timeout_total` — timeouts acumulados

**Soluções:**
1. Habilitar `leak-detection-threshold` para encontrar onde a conexão está vazando
2. Não fazer I/O externo dentro de `@Transactional` — separar a chamada HTTP da transação
3. Ajustar pool size: `connections = (cores * 2) + 1` como baseline (HikariCP wiki)
4. Revisar queries lentas com `EXPLAIN ANALYZE`

### N+1 queries (JPA/Hibernate)

**Sintoma:** endpoint que deveria fazer 1-2 queries faz 50+. Latência degrada proporcionalmente ao número de registros.

**Como detectar:**

```java
// application.yml — ver queries no log
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        generate_statistics: true  # mostra count de queries por sessão

// Ou usar Hibernate Statistics programaticamente
logging:
  level:
    org.hibernate.stat: DEBUG
```

**Cenário típico:**

```java
// RUIM — gera N+1 queries
List<Doctor> doctors = doctorRepo.findAll();  // 1 query
doctors.forEach(d -> d.getAppointments().size());  // N queries (1 por doctor)
```

**Soluções:**

```java
// 1. JOIN FETCH (JPQL)
@Query("SELECT d FROM Doctor d JOIN FETCH d.appointments WHERE d.active = true")
List<Doctor> findActiveWithAppointments();

// 2. @EntityGraph (declarativo)
@EntityGraph(attributePaths = {"appointments", "specialty"})
List<Doctor> findByActiveTrue();

// 3. @BatchSize (lazy loading em lotes, menos queries)
@Entity
public class Doctor {
    @OneToMany(mappedBy = "doctor")
    @BatchSize(size = 20)  // carrega 20 relações por vez, não 1
    private List<Appointment> appointments;
}

// 4. DTO Projection (melhor performance — sem entidade managed)
@Query("SELECT new com.app.dto.DoctorSummary(d.name, COUNT(a)) " +
       "FROM Doctor d LEFT JOIN d.appointments a GROUP BY d.name")
List<DoctorSummary> findDoctorSummaries();
```

**Regra:** sempre verificar o Hibernate statistics ou query log antes de ir para produção. N+1 em 10 registros no dev pode significar 10.000 queries em produção.

### LazyInitializationException

**Sintoma:** `org.hibernate.LazyInitializationException: could not initialize proxy - no Session`

**Causa:** acessar uma relação `@*ToMany` (lazy por default) fora do escopo de uma transação/sessão Hibernate.

```java
// RUIM — sessão já fechou quando o controller tenta serializar
@GetMapping("/doctors/{id}")
public Doctor getDoctor(@PathVariable Long id) {
    Doctor doctor = doctorRepo.findById(id).orElseThrow();
    // se Doctor tem List<Appointment> lazy, Jackson tenta serializar → BOOM
    return doctor;
}
```

**Soluções (em ordem de preferência):**

1. **DTO Projection** — retorna exatamente o que precisa, sem entidade lazy

```java
@GetMapping("/doctors/{id}")
public DoctorResponse getDoctor(@PathVariable Long id) {
    return doctorService.findById(id);  // retorna DTO, não entidade
}
```

2. **`@EntityGraph`** — carrega eager para essa query específica

```java
@EntityGraph(attributePaths = "appointments")
Optional<Doctor> findById(Long id);
```

3. **`JOIN FETCH`** — na JPQL

**Anti-pattern a evitar:** `spring.jpa.open-in-view=true` (OSIV) — mantém sessão aberta no controller. "Resolve" o erro mas causa problemas piores: connection pool exausto, queries inesperadas no view layer. **Desabilite em produção.**

```yaml
spring:
  jpa:
    open-in-view: false  # SEMPRE false em produção
```

### @Transactional: problemas sutis

**Problema 1 — `@Transactional` em método privado:**

```java
@Service
public class PaymentService {
    @Transactional  // NÃO FUNCIONA — Spring usa proxy, só intercepta public
    private void processPayment(Payment p) { ... }
}
```

Spring Boot usa proxies (CGLIB por default). O proxy só intercepta chamadas externas ao bean. Método privado ou chamada interna (self-invocation) bypassa o proxy.

**Problema 2 — self-invocation:**

```java
@Service
public class OrderService {
    public void createOrder(OrderRequest req) {
        // ... lógica
        sendConfirmation(req);  // chamada interna → @Transactional IGNORADO
    }

    @Transactional
    public void sendConfirmation(OrderRequest req) { ... }
}
```

**Solução:** extrair para outro bean, ou usar `ApplicationEventPublisher`:

```java
@Service
public class OrderService {
    private final ApplicationEventPublisher events;

    public void createOrder(OrderRequest req) {
        // ... lógica
        events.publishEvent(new OrderCreatedEvent(req));
    }
}

@Component
@TransactionalEventListener
public class OrderEventHandler {
    @Transactional
    public void onOrderCreated(OrderCreatedEvent event) { ... }
}
```

**Problema 3 — exceção checked não faz rollback:**

```java
@Transactional  // por default, só faz rollback em unchecked (RuntimeException)
public void transfer(Account from, Account to, BigDecimal amount) throws InsufficientFundsException {
    // InsufficientFundsException é checked → NÃO faz rollback!
}

// Solução: declarar explicitamente
@Transactional(rollbackFor = InsufficientFundsException.class)
```

### Memory leak e GC tuning

**Diagnóstico:**

```bash
# Heap dump quando OOM
java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heapdump.hprof -jar app.jar

# Analisar com Eclipse MAT ou VisualVM
# Buscar: objetos que crescem sem parar, collections gigantes, caches sem limite
```

**Causas comuns em Spring Boot:**
- Cache sem eviction (ex.: `@Cacheable` sem TTL → memória cresce infinitamente)
- `static` collections que acumulam dados
- Event listeners que acumulam referências
- ThreadLocal não limpo (especialmente em servidores com thread pool)

**GC tuning básico:**

```bash
# G1GC (default Java 17+) — geralmente não precisa tuning
java -Xms512m -Xmx2g -XX:+UseG1GC -jar app.jar

# ZGC (Java 21+) — low-latency, pausas < 1ms
java -Xms512m -Xmx2g -XX:+UseZGC -jar app.jar

# Métricas: monitorar via Actuator/Micrometer
management.metrics.enable.jvm=true
```

### API timeout e cascading failures

**Problema:** serviço A chama serviço B que está lento → threads de A ficam presas → A também fica lento → cascata.

**Solução com Resilience4j:**

```java
// build.gradle
implementation 'io.github.resilience4j:resilience4j-spring-boot3'

// application.yml
resilience4j:
  circuitbreaker:
    instances:
      notificationService:
        sliding-window-size: 10
        failure-rate-threshold: 50    # abre após 50% de falhas
        wait-duration-in-open-state: 30s
        permitted-number-of-calls-in-half-open-state: 3
  timelimiter:
    instances:
      notificationService:
        timeout-duration: 2s          # timeout de 2s
  retry:
    instances:
      notificationService:
        max-attempts: 3
        wait-duration: 500ms

// No service
@CircuitBreaker(name = "notificationService", fallbackMethod = "fallbackNotify")
@TimeLimiter(name = "notificationService")
@Retry(name = "notificationService")
public CompletableFuture<String> notify(NotificationRequest req) {
    return CompletableFuture.supplyAsync(() -> notificationClient.send(req));
}

public CompletableFuture<String> fallbackNotify(NotificationRequest req, Throwable t) {
    log.warn("Notification failed, queuing for retry: {}", t.getMessage());
    retryQueue.add(req);  // enfileirar para retry assíncrono
    return CompletableFuture.completedFuture("queued");
}
```

### Graceful shutdown

**Problema:** deploy mata o processo enquanto requests estão em andamento → erros 502 para o usuário.

```yaml
# application.yml
server:
  shutdown: graceful  # espera requests em andamento terminarem

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s  # tempo máximo de espera
```

```java
// Para limpeza customizada
@Component
public class CleanupHandler {
    @PreDestroy
    public void cleanup() {
        // fechar conexões, flush de buffers, etc.
        log.info("Shutting down gracefully...");
    }
}
```

### Database migrations seguras (Flyway)

**Problema:** `ALTER TABLE ADD COLUMN NOT NULL` em tabela grande trava o banco (lock de escrita).

**Migração segura em produção (expand-and-contract):**

```sql
-- V10__add_email_safe.sql
-- Step 1: adiciona coluna nullable (sem lock)
ALTER TABLE patients ADD COLUMN email VARCHAR(255);

-- V11__backfill_email.sql
-- Step 2: popula em batches (sem lock longo)
UPDATE patients SET email = 'unknown@legacy.com' WHERE email IS NULL;

-- V12__make_email_not_null.sql
-- Step 3: após deploy que já escreve email, adiciona constraint
ALTER TABLE patients ALTER COLUMN email SET NOT NULL;
```

**Regras:**
- Nunca `DROP COLUMN` na mesma release que remove o código que usa. Primeiro deploy sem o código, depois migra.
- Nunca rename de coluna em uma step — cria nova, copia, valida, depois remove antiga.
- Sempre testar migração no staging com volume real de dados.

### Distributed tracing

**Problema:** request passa por 5 microserviços, está lenta. Qual serviço é o gargalo?

```java
// build.gradle — Spring Boot 3 com Micrometer Tracing
implementation 'io.micrometer:micrometer-tracing-bridge-otel'
implementation 'io.opentelemetry:opentelemetry-exporter-otlp'

// application.yml
management:
  tracing:
    sampling:
      probability: 1.0  # 100% em dev, 10% em prod
  otlp:
    tracing:
      endpoint: http://jaeger:4318/v1/traces
```

Cada request recebe um `traceId` propagado automaticamente entre serviços via headers HTTP. Visualiza no Jaeger/Zipkin o waterfall de cada serviço e identifica o bottleneck.

## Quando usar

- **Spring Boot:** default para aplicações Java. Microserviços, APIs REST, batch processing.
- **Spring WebFlux:** quando precisa de non-blocking I/O (reactive). Útil para gateways, streaming.
- **Spring Cloud:** microserviços distribuídos (service discovery, circuit breaker, config server).
- **Spring Batch:** processamento em lote de grandes volumes.

## Armadilhas comuns

- **LazyInitializationException:** acessar relação lazy fora de transação. Resolver com `@EntityGraph`, `JOIN FETCH`, ou DTO projection.
- **N+1 queries:** ocorre quando 1 query busca N entidades e depois N queries buscam relações de cada uma (total: N+1). Detectar via query logs repetidos ou Hibernate statistics. Soluções: `JOIN FETCH` na JPQL, `@EntityGraph` na interface do repository, `@BatchSize` para lazy loading em lotes, ou DTO projection que já traz os dados necessários em 1 query.
- **`@Transactional` em private methods:** não funciona! O Spring usa proxies, que só interceptam métodos públicos.
- **Circular dependencies:** A depende de B que depende de A. Reestruturar com eventos ou extrair interface.
- **Beans mutáveis singleton:** estado compartilhado em beans singleton causa race conditions.

## Na prática (da minha experiência)

> No MedEspecialista, o backend é Spring Boot 3 com Java 21. Uso Spring Data JPA com PostgreSQL, Spring Security com JWT para autenticação, e Actuator + Micrometer para métricas exportadas ao Grafana. O CI/CD com GitHub Actions roda testes com Testcontainers (PostgreSQL e Redis em Docker), garantindo que cada PR é validada contra bancos reais. O tempo de deploy caiu de 1 hora para 2 minutos após automatizar o pipeline.

## How to explain in English

"Spring Boot is my go-to framework for Java backend development. I use it because of the mature ecosystem — Spring Data for database access, Spring Security for authentication, and Actuator for production-ready monitoring, all with minimal configuration.

My typical project structure follows a layered architecture: controllers handle HTTP mapping and validation, services contain business logic, and repositories manage data access through Spring Data JPA. I prefer constructor injection over field injection because it makes dependencies explicit and simplifies testing.

For testing, I use a combination of `@WebMvcTest` for controller unit tests, `@DataJpaTest` for repository tests, and `@SpringBootTest` with Testcontainers for full integration tests. Testcontainers is a game-changer — it spins up real PostgreSQL and Redis instances in Docker during tests, so I have high confidence that the code works correctly in production.

One area where I've seen teams struggle is with JPA's lazy loading. The LazyInitializationException is a common pitfall when accessing relationships outside of a transaction boundary. I address this by using DTO projections or `@EntityGraph` annotations, which keeps queries predictable and avoids the N+1 problem."

### Key vocabulary

- injeção de dependência → dependency injection (DI)
- contêiner IoC → IoC container: gerencia beans e suas dependências
- autoconfiguração → autoconfiguration
- perfil → profile: configuração por ambiente
- anotação → annotation: metadata no código (`@Service`, `@Repository`)
- transação → transaction: unidade atômica de trabalho no banco
- filtro de segurança → security filter chain
- inicialização tardia → lazy initialization / lazy loading

## Recursos

- [Spring Boot Reference](https://docs.spring.io/spring-boot/reference/) — documentação oficial
- [Spring Framework](https://spring.io/projects/spring-framework)
- [Spring Initializr](https://start.spring.io/) — gerador de projetos
- [Spring MVC — Web on Servlet Stack](https://docs.spring.io/spring-framework/reference/web.html)
- [Spring Security](https://spring.io/projects/spring-security)
- [Spring Testing](https://docs.spring.io/spring-framework/reference/testing.html)
- [Baeldung — Start Here](https://www.baeldung.com/start-here) — tutorials Spring Boot
- [Stop using @Autowired](https://www.linkedin.com/pulse/you-should-stop-using-spring-autowired-felix-coutinho/) — constructor injection
- [ProblemDetails para tratamento de erros](https://medium.com/@claudionetto/usando-problemdetails-para-facilitar-e-melhorar-o-retorno-de-exce%C3%A7%C3%B5es-5c060a42f637)
- [Spring @Value — Baeldung](https://www.baeldung.com/spring-value-annotation)

## Veja também

- [[Java Fundamentals]]
- [[Testes em Java]]
- [[Testes]]
- [[API Design]]
- [[Kafka]]
- [[System Design]] — troubleshooting cross-stack, building blocks
