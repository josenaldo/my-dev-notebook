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
- [Spring Initializr](https://start.spring.io/) — gerador de projetos
- [Baeldung](https://www.baeldung.com/) — tutorials Spring Boot

## Veja também

- [[Java Fundamentals]]
- [[Testes em Java]]
- [[Testes]]
- [[API Design]]
- [[Kafka]]
