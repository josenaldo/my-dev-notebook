---
title: "NestJS: fundamentos"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - nestjs
  - dependency-injection
  - modules
aliases:
  - NestJS DI
  - NestJS modules
  - "@Injectable"
---

# NestJS: fundamentos

> [!abstract] TL;DR
> NestJS é um framework TypeScript opinativo, decorator-based e com DI built-in. A unidade organizacional é o módulo: ele declara controllers, providers, imports e exports. O container resolve dependências por constructor injection. É forte para apps enterprise; é overhead em apps pequenos.

## O que é

NestJS é um framework para aplicações server-side Node que organiza código em módulos, controllers e providers. Por padrão usa Express por baixo, mas pode usar Fastify como adapter. A filosofia lembra Spring Boot/Angular: metadata via decorators, DI container e arquitetura explícita.

## Por que importa

Em apps grandes, wiring manual, lifecycle e padrões transversais viram custo. NestJS compra estrutura: módulos por feature, providers testáveis, guards/pipes/interceptors/filters e DI consistente. Em apps pequenos, a mesma estrutura pode ser mais cerimônia que benefício.

## Como funciona

```typescript
import { Module } from "@nestjs/common";
import { UsersController } from "./users.controller";
import { UsersService } from "./users.service";

@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

```typescript
import { Injectable } from "@nestjs/common";

@Injectable()
export class UsersService {
  constructor(private readonly db: DatabaseClient) {}

  async findById(id: string) {
    return this.db.users.findById(id);
  }
}
```

```typescript
import { Controller, Get, Param } from "@nestjs/common";

@Controller("users")
export class UsersController {
  constructor(private readonly users: UsersService) {}

  @Get(":id")
  async getUser(@Param("id") id: string) {
    return this.users.findById(id);
  }
}
```

```typescript
import { Injectable, Scope } from "@nestjs/common";

@Injectable({ scope: Scope.REQUEST })
export class RequestContextService {}

@Injectable({ scope: Scope.TRANSIENT })
export class NewPerInjectionService {}

// Default: singleton no módulo/app.
```

```typescript
@Module({
  imports: [DatabaseModule, UsersModule],
  controllers: [OrdersController],
  providers: [OrdersService],
  exports: [OrdersService],
})
export class OrdersModule {}
```

## Na prática

Padrão observado no ecossistema: um módulo por feature (`UsersModule`, `OrdersModule`), shared modules para concerns transversais (`DatabaseModule`, `LoggerModule`), singleton como default, request scope apenas quando precisa de contexto da request. `exports` é o contrato entre módulos.

## Armadilhas

1. `Scope.REQUEST` usado sem necessidade: escopo request propaga para dependências e pode custar performance.
2. Circular imports entre módulos: `forwardRef()` existe, mas frequentemente sinaliza design ruim.
3. Esquecer `exports`: outro módulo importa mas não consegue injetar o provider.
4. Colocar lógica pesada no constructor: prefira lifecycle hooks como `OnModuleInit`.

## Em entrevista

"NestJS is opinionated and decorator-based, with a built-in dependency injection container. Its fundamental unit is the module: a module declares controllers, providers, imports, and exports. Providers default to singleton scope; request and transient scopes exist but should be used deliberately. It is a good fit for enterprise apps with complex domains and teams that benefit from structure."

Vocabulário-chave:

- dependency injection -> injeção de dependência
- provider -> componente injetável
- module -> módulo
- controller -> controlador HTTP
- scope -> escopo

## Fontes

- [NestJS docs](https://docs.nestjs.com/)

## Veja também

- [[01 - Os 4 frameworks - Express, NestJS, Fastify, Hono]]
- [[04 - NestJS - guards, interceptors, pipes, filters]]
- [[09 - Validation com schema]]
- [[11 - DI - manual vs container]]
- [[Node.js]]

