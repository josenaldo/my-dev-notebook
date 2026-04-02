---
title: "Spring Boot"
created: 2026-04-01
updated: 2026-04-01
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

## Quando usar

- **Spring Boot:** default para aplicações Java. Microserviços, APIs REST, batch processing.
- **Spring WebFlux:** quando precisa de non-blocking I/O (reactive). Útil para gateways, streaming.
- **Spring Cloud:** microserviços distribuídos (service discovery, circuit breaker, config server).
- **Spring Batch:** processamento em lote de grandes volumes.

## Armadilhas comuns

- **LazyInitializationException:** acessar relação lazy fora de transação. Resolver com `@EntityGraph`, `JOIN FETCH`, ou DTO projection.
- **N+1 queries:** loop de consultas dentro de `@Transactional`. Usar `@EntityGraph` ou `@BatchSize`.
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
