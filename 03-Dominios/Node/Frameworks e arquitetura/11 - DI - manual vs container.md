---
title: "DI: manual vs container"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - dependency-injection
  - di
  - architecture
aliases:
  - DI manual
  - DI container
  - tsyringe
  - awilix
---

# DI: manual vs container

> [!abstract] TL;DR
> DI manual é constructor injection + composition root. Containers (NestJS, awilix, tsyringe, InversifyJS) automatizam wiring e lifecycle scopes. Manual é simples e explícito em apps pequenos; container vale quando o grafo de dependências e escopos fica complexo.

## O que é

Dependency Injection é passar dependências de fora para dentro, em vez de instanciá-las dentro da classe/função. Container de DI é uma ferramenta que resolve esse grafo automaticamente.

## Por que importa

Sem DI, use cases criam DB clients, loggers e gateways diretamente. Isso acopla, dificulta teste e viola [[10 - Clean Architecture em Node]]. Com DI, dependências ficam explícitas. Com container, o wiring pode ser centralizado e escalável.

## Como funciona

```typescript
// composition-root.ts
const logger = new Logger();
const db = new PgPool(config.database);
const userRepo = new UserRepositoryPg(db);
const createUser = new CreateUserUseCase(userRepo, logger);
const userController = new UserController(createUser);

export { userController };
```

```typescript
export function makeUserController(deps: { db: PgPool; logger: Logger }) {
  const repo = new UserRepositoryPg(deps.db);
  const createUser = new CreateUserUseCase(repo, deps.logger);
  return new UserController(createUser);
}
```

```typescript
@Injectable()
export class CreateUserUseCase {
  constructor(
    private readonly repo: UserRepository,
    private readonly logger: Logger,
  ) {}
}
```

```typescript
import { asClass, asValue, createContainer, InjectionMode } from "awilix";

const container = createContainer({ injectionMode: InjectionMode.PROXY });
container.register({
  db: asValue(pgPool),
  logger: asClass(Logger).singleton(),
  userRepo: asClass(UserRepositoryPg).singleton(),
  createUser: asClass(CreateUserUseCase).singleton(),
});

const createUser = container.resolve<CreateUserUseCase>("createUser");
```

```typescript
import "reflect-metadata";
import { container, inject, injectable } from "tsyringe";

@injectable()
class CreateUserUseCase {
  constructor(
    @inject("UserRepository") private readonly repo: UserRepository,
    @inject("Logger") private readonly logger: Logger,
  ) {}
}

container.register("UserRepository", { useClass: UserRepositoryPg });
const useCase = container.resolve(CreateUserUseCase);
```

| Approach | Ramp-up | Scopes | Quando usar |
| --- | --- | --- | --- |
| Manual | Baixo | Manual | App pequeno, dependências rasas |
| NestJS DI | Médio | Singleton/request/transient | Apps NestJS |
| awilix | Médio | Singleton/scoped/transient | Express/Fastify com grafo médio |
| tsyringe | Médio | Decorator-based | Times confortáveis com decorators |
| InversifyJS | Alto | Completo | Apps grandes, legado com container |

## Na prática

Até cerca de poucas dezenas de services, composition root manual costuma ser suficiente. Quando há módulos, request-scoped providers, lazy loading ou muitos adapters, container paga o custo. O ponto não é "container moderno"; é custo de wiring versus clareza.

## Armadilhas

1. Container em app pequeno por antecipação: complexidade antes da necessidade.
2. Misturar DI manual e container sem fronteira: ninguém sabe quem instancia quem.
3. `tsyringe` sem `reflect-metadata`: falhas em runtime.
4. Request scope viral em NestJS: dependências podem virar request-scoped.
5. Composition root espalhado: wiring perde rastreabilidade.

## Em entrevista

"Dependency injection without a container is constructor injection plus a composition root. You instantiate dependencies explicitly at startup and pass them in. Containers like NestJS DI, awilix, and tsyringe automate wiring and lifecycle scopes. Manual DI is better for small apps; containers earn their complexity in large apps, complex domains, or request-scoped lifecycles."

Vocabulário-chave:

- composition root -> raiz de composição
- constructor injection -> injeção por construtor
- container -> contêiner de DI
- lifecycle scope -> escopo de ciclo de vida
- request-scoped -> escopado por request

## Fontes

- [awilix](https://github.com/jeffijoe/awilix)
- [tsyringe](https://github.com/microsoft/tsyringe)
- [NestJS docs](https://docs.nestjs.com/)

## Veja também

- [[03 - NestJS - fundamentos]]
- [[10 - Clean Architecture em Node]]
- [[12 - Decision tree + cheatsheet]]
- [[Node.js]]

