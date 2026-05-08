---
title: "Validation com schema"
created: 2026-05-08
updated: 2026-05-08
type: concept
status: seedling
publish: true
tags:
  - node
  - frameworks
  - validation
  - zod
  - schema
aliases:
  - zod
  - JSON Schema
  - ValidationPipe
  - class-validator
---

# Validation com schema

> [!abstract] TL;DR
> Schema-first é o padrão pragmático para validation em Node moderno. `zod` une schema, runtime validation e type inference; Fastify usa JSON Schema nativo; NestJS usa `ValidationPipe` com `class-validator` ou wrappers zod. O objetivo é evitar validação manual espalhada.

## O que é

Validation com schema define o contrato de input em um objeto reutilizável. Esse contrato valida dados em runtime e, dependendo da ferramenta, também gera tipos TypeScript e documentação.

## Por que importa

Toda entrada externa é `unknown`: body, query, params, headers, webhooks, mensagens de fila. Sem schema, validação vira if espalhado em controller. Com schema, validação fica centralizada, testável e reaproveitável.

## Como funciona

```typescript
import { z } from "zod";

const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).optional(),
}).strict();

type CreateUser = z.infer<typeof CreateUserSchema>;

app.post("/users", (req, res) => {
  const data: CreateUser = CreateUserSchema.parse(req.body);
  res.status(201).json(data);
});
```

```typescript
app.post(
  "/users",
  {
    schema: {
      body: {
        type: "object",
        required: ["name", "email"],
        additionalProperties: false,
        properties: {
          name: { type: "string", minLength: 1, maxLength: 100 },
          email: { type: "string", format: "email" },
          age: { type: "integer", minimum: 0 },
        },
      },
    },
  },
  async (req) => req.body,
);
```

```typescript
import { IsEmail, IsInt, IsOptional, IsString, MaxLength, Min, MinLength } from "class-validator";

export class CreateUserDto {
  @IsString()
  @MinLength(1)
  @MaxLength(100)
  name!: string;

  @IsEmail()
  email!: string;

  @IsOptional()
  @IsInt()
  @Min(0)
  age?: number;
}
```

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }),
);
```

```typescript
import { zValidator } from "@hono/zod-validator";

app.post("/users", zValidator("json", CreateUserSchema), (c) => {
  const data = c.req.valid("json");
  return c.json(data, 201);
});
```

## Na prática

`zod` é forte em projetos TypeScript por type inference e ergonomia. Fastify brilha quando JSON Schema já é o contrato da API e OpenAPI importa. NestJS clássico usa DTO class + decorators; projetos novos também podem padronizar zod com wrapper. `joi` ainda aparece em legacy, mas não é a escolha natural para TS-first.

## Armadilhas

1. Schema permissivo sem `.strict()` ou `additionalProperties: false`.
2. Validar só body e esquecer query/params/headers.
3. Misturar zod e `class-validator` sem convenção clara.
4. Mensagens de erro cruas demais para cliente final.
5. Validar depois do use case: domínio já recebeu dado inválido.

## Em entrevista

"Schema-first validation is the modern Node pattern. With zod, you define a schema once and get both runtime validation and TypeScript inference through `z.infer`. Fastify uses JSON Schema natively for validation and serialization. NestJS traditionally uses DTO classes with `class-validator` through a global `ValidationPipe`, though zod wrappers are common. The senior pattern is validating every external boundary, not only request bodies."

Vocabulário-chave:

- schema-first -> orientado por schema
- type inference -> inferência de tipo
- JSON Schema -> schema JSON
- ValidationPipe -> pipe de validação
- external boundary -> fronteira externa

## Fontes

- [Zod](https://zod.dev/)
- [Fastify validation and serialization](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/)
- [NestJS docs](https://docs.nestjs.com/)

## Veja também

- [[04 - NestJS - guards, interceptors, pipes, filters]]
- [[05 - Fastify - schema-first, plugins, performance]]
- [[06 - Hono e edge runtimes]]
- [[08 - Error handling estruturado]]
- [[Node.js]]

