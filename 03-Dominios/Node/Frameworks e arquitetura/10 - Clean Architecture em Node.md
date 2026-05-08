---
title: "Clean Architecture em Node"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - clean-architecture
  - hexagonal
  - architecture
aliases:
  - Clean Architecture
  - Hexagonal
  - Ports and adapters
---

# Clean Architecture em Node

> [!abstract] TL;DR
> Clean Architecture organiza código em camadas: entities, use cases, interface adapters e frameworks/drivers. A dependency rule diz que dependências apontam para dentro. Em Node, isso aparece como domínio sem Express/ORM, use cases dependentes de ports e adapters externos. Use quando domínio justifica.

## O que é

Clean Architecture, popularizada por Robert C. Martin, separa políticas de negócio de mecanismos externos. Hexagonal/Ports and Adapters é uma formulação próxima: domínio e aplicação definem portas; infraestrutura implementa adapters.

## Por que importa

Se use case importa `Request` do Express ou entity importa decorator do ORM, framework virou parte do núcleo. Isso dificulta teste, troca de adapter e manutenção. Em domínio rico, separar camadas reduz acoplamento. Em CRUD simples, pode ser over-engineering.

## Como funciona

```text
Entities
└── Use Cases
    └── Interface Adapters
        └── Frameworks and Drivers
```

```text
src/
├── domain/
│   └── User.ts
├── application/
│   ├── CreateUserUseCase.ts
│   └── UserRepository.ts
├── infrastructure/
│   ├── db/UserRepositoryPg.ts
│   └── http/UserController.ts
└── presentation/
    └── server.ts
```

```typescript
export class User {
  constructor(
    public readonly id: string,
    public name: string,
    public readonly email: string,
  ) {}

  changeName(name: string) {
    if (name.length < 1) throw new Error("Name required");
    this.name = name;
  }
}
```

```typescript
export interface UserRepository {
  save(user: User): Promise<void>;
  findById(id: string): Promise<User | null>;
}

export class CreateUserUseCase {
  constructor(private readonly repo: UserRepository) {}

  async execute(input: { name: string; email: string }) {
    const user = new User(crypto.randomUUID(), input.name, input.email);
    await this.repo.save(user);
    return user;
  }
}
```

```typescript
export class UserRepositoryPg implements UserRepository {
  constructor(private readonly db: PgPool) {}

  async save(user: User): Promise<void> {
    await this.db.query("insert into users(id, name, email) values($1, $2, $3)", [
      user.id,
      user.name,
      user.email,
    ]);
  }

  async findById(id: string): Promise<User | null> {
    const result = await this.db.query("select * from users where id = $1", [id]);
    return result.rowCount ? new User(result.rows[0].id, result.rows[0].name, result.rows[0].email) : null;
  }
}
```

```typescript
const repo = new UserRepositoryPg(pgPool);
const useCase = new CreateUserUseCase(repo);
const controller = new UserController(useCase);

app.post("/users", (req, res) => controller.create(req, res));
```

## Na prática

Use Clean quando há regra de negócio rica, múltiplos adapters, testes de use case valiosos e vida longa do produto. Em Express/Fastify, composition root manual faz o wiring. Em NestJS, módulos e providers ajudam, mas não garantem Clean: ainda é possível violar camadas.

## Armadilhas

1. Entity importando ORM/decorator: framework entrou no domínio.
2. Use case recebendo `Request`/`Response`: framework entrou na aplicação.
3. Aplicar Clean em CRUD trivial: complexidade sem retorno.
4. Adapter virando god class: separar por bounded context/feature.
5. Sem composition root claro: dependências ficam espalhadas.

## Em entrevista

"Clean Architecture separates business policies from external mechanisms. The dependency rule is the key: source dependencies point inward, so the domain does not know about Express, databases, or framework details. In Node, that means use cases depend on ports, and infrastructure implements adapters. It is valuable for complex domains, but over-engineering for simple CRUD."

Vocabulário-chave:

- layer -> camada
- dependency rule -> regra de dependência
- port -> porta
- adapter -> adaptador
- dependency inversion -> inversão de dependência

## Fontes

- [The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

## Veja também

- [[03 - NestJS - fundamentos]]
- [[08 - Error handling estruturado]]
- [[09 - Validation com schema]]
- [[11 - DI - manual vs container]]
- [[Node.js]]
- [[Arquitetura de Software]]

