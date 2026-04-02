---
title: "API Design"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - arquitetura
  - entrevista
publish: false
---

# API Design

A arte de projetar interfaces de comunicação entre sistemas que sejam intuitivas, consistentes e evoluíveis.

## O que é

API Design é o processo de definir como sistemas se comunicam — quais endpoints existem, que dados trafegam, como erros são reportados, e como a API evolui sem quebrar clientes existentes. Para um senior fullstack, é uma skill essencial tanto no backend (construir APIs) quanto no frontend (consumir APIs).

## Como funciona

### REST — Princípios

**Recursos como substantivos:** URLs representam entidades, não ações.

```text
✅ GET /api/patients/123/appointments
❌ GET /api/getPatientAppointments?patientId=123
```

**Verbos HTTP corretos:**

| Método | Uso | Idempotente |
| --- | --- | --- |
| GET | Buscar recurso(s) | Sim |
| POST | Criar recurso | Não |
| PUT | Substituir recurso completo | Sim |
| PATCH | Atualizar parcialmente | Não* |
| DELETE | Remover recurso | Sim |

\* PATCH pode ser idempotente dependendo da implementação.

**Status codes semânticos:**

| Range | Significado | Exemplos |
| --- | --- | --- |
| 2xx | Sucesso | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirecionamento | 301 Moved, 304 Not Modified |
| 4xx | Erro do cliente | 400 Bad Request, 401 Unauthorized, 404 Not Found, 422 Unprocessable |
| 5xx | Erro do servidor | 500 Internal, 502 Bad Gateway, 503 Unavailable |

### Paginação

```text
GET /api/patients?page=2&size=20&sort=name,asc

Response:
{
  "content": [...],
  "page": 2,
  "size": 20,
  "totalElements": 150,
  "totalPages": 8
}
```

Alternativa: **cursor-based pagination** para datasets grandes ou que mudam frequentemente.

### Versionamento

- **URL path:** `/api/v1/patients` — mais comum, mais explícito
- **Header:** `Accept: application/vnd.api+json;version=1` — mais correto, menos prático
- **Query param:** `/api/patients?version=1` — menos usado

### Tratamento de erros — RFC 9457

```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Error",
  "status": 422,
  "detail": "O campo 'email' é obrigatório",
  "instance": "/api/patients/123"
}
```

Padronizar erros com Problem Details (RFC 9457) facilita o consumo por clientes e debugging.

### REST vs GraphQL vs gRPC

**REST:** default para APIs públicas. Bem suportado, cacheável (HTTP cache, CDN), ecosistema maduro (OpenAPI/Swagger).

**GraphQL:** quando clientes diferentes precisam de dados diferentes. Evita over-fetching e under-fetching. Mais complexo no backend (resolvers, N+1, rate limiting por campo).

**gRPC:** comunicação entre microserviços internos. Protocol Buffers são menores que JSON. HTTP/2 com multiplexing. Streaming bidirecional.

### Boas práticas

- **HATEOAS:** incluir links para ações relacionadas na resposta. Na prática, poucas APIs implementam completamente.
- **Rate Limiting:** proteger a API com headers `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`.
- **Idempotency Keys:** para operações POST, o cliente envia um header `Idempotency-Key` (geralmente UUID v4) para evitar duplicação. O servidor armazena o resultado da primeira requisição e retorna o mesmo resultado em retentativas — incluindo erros 500. Padrão popularizado pela Stripe, essencial para pagamentos e criação de recursos. Keys devem ter TTL (Stripe: 24h).
- **Backward Compatibility:** nunca remover campos ou mudar tipos em versões existentes. Adicionar é safe.

> **Fontes sobre idempotência:**
> - [Stripe — Idempotent Requests](https://docs.stripe.com/api/idempotent_requests)
> - [Idempotency in Payment Processing](https://www.computer.org/publications/tech-news/trends/idempotency-in-payment-processing-architecture)
- **Documentation:** OpenAPI/Swagger para REST, esquema para GraphQL, `.proto` files para gRPC.

## Quando usar

- **REST:** APIs públicas, CRUD, sistemas que precisam de cache HTTP
- **GraphQL:** clientes com necessidades de dados muito variadas (mobile vs web vs admin)
- **gRPC:** comunicação interna entre microserviços, streaming, alta performance

## Armadilhas comuns

- **Verbos na URL:** `/api/createUser` em vez de `POST /api/users`
- **Status code genérico:** retornar 200 para tudo com `{ "success": false }` dentro do body
- **Não paginar:** retornar coleções inteiras sem paginação → timeout, OOM
- **Quebrar contratos:** remover campos, mudar tipos, alterar comportamento sem versionar
- **PUT vs PATCH:** usar PUT para atualizações parciais (deveria enviar o recurso completo)

## Na prática (da minha experiência)

> Em projetos Spring Boot, uso Spring MVC com ResponseEntity para controle fino de status codes e headers. Para documentação, configuro SpringDoc (OpenAPI 3) que gera Swagger UI automaticamente a partir das annotations. No MedEspecialista, implementei tratamento de erros padronizado com RFC 9457 usando `@ControllerAdvice`, o que simplificou o debugging tanto no backend quanto no frontend React.

## How to explain in English

"API design is about creating contracts that are intuitive for consumers and maintainable for producers. I follow REST principles by default — resources as nouns, proper HTTP verbs, semantic status codes, and consistent error responses using RFC 9457 Problem Details.

One thing I always emphasize is backward compatibility. Once an API is published, changing it can break clients. I use URL-based versioning for major changes and only add new fields — never remove or rename existing ones — within a version.

For pagination, I prefer cursor-based pagination for real-time data where records can be inserted between pages, and offset-based for static datasets where users need to jump to specific pages. The choice depends on the data characteristics and client needs.

When comparing REST, GraphQL, and gRPC, I choose based on the use case. REST for public APIs because of caching, tooling, and familiarity. GraphQL when mobile clients need minimal data transfers. And gRPC for internal microservice communication where binary serialization and HTTP/2 streaming give us a significant performance advantage."

### Key vocabulary

- endpoint → endpoint: URL que aceita requisições
- recurso → resource: entidade representada pela API
- idempotente → idempotent: mesma requisição produz mesmo resultado
- paginação → pagination
- versionamento → versioning
- contrato → contract: especificação da API
- compatibilidade retroativa → backward compatibility
- documentação da API → API documentation (OpenAPI/Swagger)

## Recursos

- [Best Practices for Designing a Pragmatic RESTful API](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
- [REST API Best Practices](https://www.freecodecamp.org/news/rest-api-best-practices-rest-endpoint-design-examples/)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)
- [HTTP PUT vs HTTP PATCH](https://www.baeldung.com/http-put-patch-difference-spring)
- [Using JSON Patch in Spring REST APIs](https://www.baeldung.com/spring-rest-json-patch#use-case)

> [!info] REST do Jeito Certo
> [https://www.youtube.com/playlist?list=PLkpjQs-GfEMPituMCb77qd90onpF3khFt](https://www.youtube.com/playlist?list=PLkpjQs-GfEMPituMCb77qd90onpF3khFt)

## Veja também

- [[Redes e Protocolos]]
- [[System Design]]
- [[Spring Boot]]
- [[Arquitetura de Software]]
