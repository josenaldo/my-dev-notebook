---
title: "Error handling estruturado"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - error-handling
  - problem-details
  - rfc-7807
aliases:
  - Problem Details
  - RFC 7807
  - error middleware
---

# Error handling estruturado

> [!abstract] TL;DR
> Problem Details (RFC 7807) padroniza erros HTTP com `application/problem+json` e campos como `type`, `title`, `status`, `detail`, `instance`. Express implementa com error middleware; NestJS com exception filter; Fastify com `setErrorHandler`; Hono com `app.onError`.

## O que é

Error handling estruturado é tratar erro como contrato de API. Em vez de cada endpoint retornar JSON diferente, a API usa um envelope previsível que clientes e logs conseguem entender.

## Por que importa

Sem padrão, cliente faz parsing ad-hoc, observability perde contexto e bugs 4xx/5xx se misturam. Com Problem Details, cada erro tem status, título, detalhe e instância da request.

## Como funciona

```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Failed",
  "status": 400,
  "detail": "Field 'email' must be a valid email",
  "instance": "/api/v1/users"
}
```

```typescript
class HttpError extends Error {
  constructor(
    public readonly status: number,
    public readonly type: string,
    message: string,
  ) {
    super(message);
  }
}

app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  if (res.headersSent) return next(err);
  const status = err instanceof HttpError ? err.status : 500;
  res.status(status).type("application/problem+json").json({
    type: err instanceof HttpError ? err.type : "about:blank",
    title: err.name,
    status,
    detail: status >= 500 ? "Unexpected error" : err.message,
    instance: req.originalUrl,
  });
});
```

```typescript
@Catch()
export class ProblemDetailsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const res = ctx.getResponse<Response>();
    const req = ctx.getRequest<Request>();
    const status = exception instanceof HttpException ? exception.getStatus() : 500;

    res.status(status).type("application/problem+json").json({
      type: "about:blank",
      title: "Error",
      status,
      detail: status >= 500 ? "Unexpected error" : String(exception),
      instance: req.url,
    });
  }
}
```

```typescript
app.setErrorHandler((err, req, reply) => {
  const status = err.statusCode ?? 500;
  reply.code(status).type("application/problem+json").send({
    type: "about:blank",
    title: err.name,
    status,
    detail: status >= 500 ? "Unexpected error" : err.message,
    instance: req.url,
  });
});
```

```typescript
app.onError((err, c) => {
  const status = err instanceof HTTPException ? err.status : 500;
  return c.json(
    { type: "about:blank", title: "Error", status, detail: err.message, instance: c.req.url },
    status,
    { "Content-Type": "application/problem+json" },
  );
});
```

Taxonomy prática:

- 4xx: cliente errou (validation, auth, not found).
- 5xx: servidor errou (bug, dependência indisponível, timeout interno).
- Programmer error: bug; logar e responder 500 genérico.
- Operational error: esperado; responder status específico.

## Na prática

Use handler global. Não exponha stack trace em produção. Inclua correlation ID via header/campo extra quando existir. Faça classes/tipos de erro pequenos: `ValidationError`, `NotFoundError`, `UnauthorizedError`, `ConflictError`.

## Armadilhas

1. Stack trace em produção: vazamento de informação.
2. Express error middleware com 3 args: não captura erro.
3. Responder 500 para validação ou 400 para DB down: taxonomy quebrada.
4. Handler async sem wrapper em Express 4: rejeição não chega ao handler global.
5. `detail` com mensagem interna de banco: pode vazar schema ou credenciais.

## Em entrevista

"Problem Details, from RFC 7807, gives HTTP APIs a standard error envelope: `type`, `title`, `status`, `detail`, and `instance`, usually with `application/problem+json`. Express uses error middleware, NestJS uses exception filters, Fastify uses `setErrorHandler`, and Hono uses `app.onError`. The senior part is taxonomy: operational errors get specific 4xx/5xx responses, programmer errors get logged and sanitized."

Vocabulário-chave:

- Problem Details -> detalhes de problema
- error middleware -> middleware de erro
- exception filter -> filtro de exceção
- error taxonomy -> taxonomia de erros
- correlation ID -> identificador de correlação

## Fontes

- [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807)
- [Express error handling](https://expressjs.com/en/guide/error-handling.html)

## Veja também

- [[02 - Express idiomático]]
- [[04 - NestJS - guards, interceptors, pipes, filters]]
- [[05 - Fastify - schema-first, plugins, performance]]
- [[09 - Validation com schema]]
- [[Node.js]]

