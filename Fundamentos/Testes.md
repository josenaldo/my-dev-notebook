---
title: "Testes"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - fundamentos
  - entrevista
publish: false
---

# Testes

Práticas e ferramentas para garantir que o software funciona como esperado e continua funcionando após mudanças.

## O que é

Testes automatizados são a rede de segurança de um sistema. Em entrevistas de senior, espera-se conhecimento não apenas de como escrever testes, mas de **estratégia de testes** — quais tipos usar, quando, e como equilibrar coverage com custo de manutenção.

## Como funciona

### Pirâmide de testes

```text
        /  E2E  \         ← poucos, lentos, frágeis
       /----------\
      / Integração \      ← moderados, testam contratos
     /--------------\
    /    Unitários    \   ← muitos, rápidos, isolados
```

- **Unitários:** testam uma unidade (função, classe) isolada. Rápidos, determinísticos. Mock de dependências.
- **Integração:** testam a interação entre componentes reais (API + banco, serviço + fila). Mais lentos, mas mais confiança.
- **E2E (End-to-End):** testam o fluxo completo do usuário. Frágeis e lentos, mas validam o sistema como um todo.

### Padrão AAA (Arrange-Act-Assert)

```java
// Arrange
var user = new User("Josenaldo", "josenaldo@email.com");

// Act
var result = userService.register(user);

// Assert
assertThat(result.isActive()).isTrue();
```

### Test Doubles

| Tipo | O que faz | Quando usar |
| --- | --- | --- |
| Mock | Simula comportamento e verifica chamadas | Testar interações com dependências |
| Stub | Retorna valores pré-definidos | Controlar cenários específicos |
| Spy | Wrapper real que registra chamadas | Verificar sem substituir comportamento |
| Fake | Implementação funcional simplificada | In-memory database, fake server |

### TDD (Test-Driven Development)

1. **Red:** escrever teste que falha
2. **Green:** implementar o mínimo para passar
3. **Refactor:** melhorar o código mantendo testes verdes

**Quando aplicar TDD:** lógica de negócio complexa, algoritmos, APIs com contrato definido. Não precisa aplicar em tudo — CRUD simples pode ser testado depois.

### Ferramentas por stack

**Java / Spring Boot:**

- JUnit 5 — framework base
- AssertJ — assertions fluentes e legíveis
- Mockito — mocking framework
- Testcontainers — bancos de dados reais em Docker para testes de integração
- MockMvc — testes de controllers HTTP
- JsonPath / JSONassert — validação de respostas JSON
- Awaitility — testes assíncronos

**JavaScript / TypeScript:**

- Jest — framework completo (runner + assertions + mocks)
- Vitest — alternativa rápida, compatível com Vite
- Testing Library — testes de componentes React focados em comportamento do usuário
- Playwright / Cypress — testes E2E

## Quando usar

- **Unitários:** lógica de negócio, algoritmos, transformações de dados, validações
- **Integração:** endpoints de API, queries de banco, interação com filas/cache
- **E2E:** fluxos críticos de negócio (checkout, login, onboarding)
- **Contract tests:** quando microserviços dependem uns dos outros (Pact, Spring Cloud Contract)

## Armadilhas comuns

- **Testar implementação, não comportamento:** testes que verificam métodos internos quebram a cada refactor. Teste o "o quê", não o "como".
- **Over-mocking:** se precisa mockar 5 dependências, o código provavelmente tem responsabilidades demais.
- **Testes flaky:** testes que falham intermitentemente. Causas comuns: dependência de ordem, timing, estado compartilhado.
- **Coverage como métrica única:** 100% coverage não significa código testado. Testes ruins com alta coverage dão falsa confiança.
- **Não testar edge cases:** input vazio, null, limites de range, concorrência.

## Na prática (da minha experiência)

> Em projetos Spring Boot, uso uma combinação de testes unitários com Mockito para lógica de negócio e testes de integração com Testcontainers para verificar queries e endpoints. No MedEspecialista, implementei CI/CD com GitHub Actions que roda a suíte de testes a cada push, reduzindo o tempo de deploy de 1 hora para 2 minutos. A regra é: se não tem teste, não vai pro main.

## How to explain in English

"My testing strategy follows the testing pyramid — many fast unit tests, a moderate number of integration tests, and a few critical E2E tests. The goal is fast feedback loops: unit tests catch logic errors in seconds, integration tests catch wiring issues in minutes, and E2E tests validate the most critical user flows.

In Spring Boot projects, I use JUnit 5 with AssertJ for readable assertions and Mockito for isolation. For integration tests, I use Testcontainers to spin up real PostgreSQL and Redis instances in Docker — this gives me confidence that my queries and configurations work correctly without maintaining a separate test database.

I'm pragmatic about TDD. I use it when writing complex business logic because it forces me to think about the API before the implementation. But for straightforward CRUD operations, I typically write the code first and tests after, focusing on edge cases and error scenarios.

One thing I always emphasize is testing behavior, not implementation. My tests describe what the system should do, not how it does it internally. This means they survive refactoring — I can restructure the code without rewriting the tests."

### Key vocabulary

- teste unitário → unit test: testa uma unidade isolada
- teste de integração → integration test: testa interação entre componentes
- teste ponta a ponta → end-to-end test (E2E)
- cobertura de código → code coverage
- objeto simulado → mock object
- desenvolvimento guiado por testes → test-driven development (TDD)
- teste intermitente → flaky test
- suíte de testes → test suite

## Recursos

- [Testing Trophy — Kent C. Dodds](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications)
- [Testcontainers](https://testcontainers.com/) — bancos reais em Docker para testes

> [!info] Aplicando Arquitetura Hexagonal com testes de integração e unidade
> [https://www.youtube.com/live/DWsxTJpxaOo?si=Pdtll7mDoe9wQJ9z](https://www.youtube.com/live/DWsxTJpxaOo?si=Pdtll7mDoe9wQJ9z)

## Veja também

- [[Testes em Java]]
- [[Orientação a Objetos]]
- [[Spring Boot]]
