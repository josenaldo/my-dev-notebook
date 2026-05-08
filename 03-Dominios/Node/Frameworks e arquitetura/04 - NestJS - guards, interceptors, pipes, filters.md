---
title: "NestJS: guards, interceptors, pipes, filters"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - nestjs
  - guards
  - interceptors
  - pipes
  - filters
aliases:
  - NestJS lifecycle
  - "@UseGuards"
  - "@UseInterceptors"
  - ValidationPipe
---

# NestJS: guards, interceptors, pipes, filters

> [!abstract] TL;DR
> NestJS separa concerns no lifecycle da request: Guards decidem se a request passa; Pipes validam/transformam input; Interceptors envolvem o handler antes/depois; Exception Filters formatam erros. Cada hook pode ser aplicado por rota, controller ou globalmente.

## O que é

Esses quatro hooks são o motivo de NestJS não ser apenas routing com decorators. Eles transformam auth, validation, logging, cache, timeout e error handling em componentes reutilizáveis.

Fluxo mental:

```text
Middleware -> Guard -> Interceptor(before) -> Pipe -> Handler
                    -> Interceptor(after) -> Exception Filter(se erro)
```

## Por que importa

Sem lifecycle hooks, controllers acumulam boilerplate: checar auth, validar body, medir tempo, transformar resposta e formatar erro em cada método. Em NestJS idiomático, controller coordena caso de uso; concerns transversais ficam nos hooks certos.

## Como funciona

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(ctx: ExecutionContext): boolean {
    const req = ctx.switchToHttp().getRequest<Request>();
    return Boolean(req.headers.authorization);
  }
}

@UseGuards(AuthGuard)
@Get("profile")
getProfile() {
  return this.users.current();
}
```

```typescript
@UsePipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }))
@Post("users")
createUser(@Body() dto: CreateUserDto) {
  return this.users.create(dto);
}
```

```typescript
@Injectable()
export class TimingInterceptor implements NestInterceptor {
  intercept(_ctx: ExecutionContext, next: CallHandler) {
    const start = Date.now();
    return next.handle().pipe(
      tap(() => console.log(`request took ${Date.now() - start}ms`)),
    );
  }
}
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
      detail: exception instanceof Error ? exception.message : "Unexpected error",
      instance: req.url,
    });
  }
}
```

```typescript
// main.ts
app.useGlobalGuards(new AuthGuard());
app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
app.useGlobalInterceptors(new TimingInterceptor());
app.useGlobalFilters(new ProblemDetailsFilter());
```

## Na prática

- Auth/autorização: Guard global ou por feature.
- Validation: `ValidationPipe` global com `whitelist` e `forbidNonWhitelisted`.
- Logging/timing/cache: Interceptor.
- Problem Details: Exception Filter global.
- Controller: fino, chamando use case/service.

## Armadilhas

1. Confundir Guard com Pipe: Guard autoriza; Pipe transforma/valida input.
2. Interceptor com trabalho CPU-heavy dentro de `tap`: bloqueia event loop.
3. Filter `@Catch()` genérico demais sem taxonomy: todos os erros viram iguais.
4. DTO sem decorators de validação: `ValidationPipe` não tem o que validar.
5. Aplicar hook global sem pensar em rotas públicas: auth pode bloquear health check.

## Em entrevista

"NestJS request lifecycle has specialized hooks. Guards decide whether a request is allowed to proceed. Pipes validate and transform input, with `ValidationPipe` being the canonical example. Interceptors wrap the handler with before-and-after behavior, useful for logging, caching, and response transformation. Exception filters format errors, often as Problem Details. Each can be route-scoped, controller-scoped, or global."

Vocabulário-chave:

- guard -> guarda de autorização
- pipe -> validação/transformação
- interceptor -> wrapper de execução
- exception filter -> filtro de exceção
- lifecycle hook -> gancho do ciclo de vida

## Fontes

- [NestJS docs](https://docs.nestjs.com/)

## Veja também

- [[03 - NestJS - fundamentos]]
- [[07 - Middleware pipeline]]
- [[08 - Error handling estruturado]]
- [[09 - Validation com schema]]
- [[Node.js]]

