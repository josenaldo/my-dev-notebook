---
title: "TypeScript"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - javascript
  - typescript
  - entrevista
publish: false
---

# TypeScript

Superset de JavaScript com tipagem estática — a ponte entre a flexibilidade do JS e a segurança de tipos do Java.

## O que é

TypeScript adiciona tipos estáticos ao JavaScript, checados em compile-time e removidos em runtime (type erasure, similar a Java Generics). Para entrevistas, o foco é em: type system, generics, utility types, type narrowing, e quando tipos são valiosos vs overhead.

## Como funciona

### Tipos básicos

```typescript
// Primitivos
let name: string = "Josenaldo";
let age: number = 40;
let active: boolean = true;

// Arrays
let ids: number[] = [1, 2, 3];
let names: Array<string> = ["a", "b"];

// Objetos
interface Patient {
  id: string;
  name: string;
  email?: string;           // opcional
  readonly createdAt: Date;  // imutável
}

// Union types
type Status = "active" | "inactive" | "suspended";

// Intersection types
type AdminUser = User & { permissions: string[] };
```

### Interface vs Type

| Aspecto | Interface | Type |
| --- | --- | --- |
| Extensão | `extends` (merge automático) | `&` (intersection) |
| Declaration merging | Sim | Não |
| Union/Intersection | Não | Sim |
| Computed properties | Não | Sim |

**Regra prática:** `interface` para contratos de objetos e classes; `type` para unions, tuples, e tipos complexos.

### Generics

```typescript
// Função genérica
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

// Com constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Generic interface
interface Repository<T> {
  findById(id: string): Promise<T>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}
```

### Utility Types

```typescript
Partial<T>      // Todos os campos opcionais
Required<T>     // Todos os campos obrigatórios
Readonly<T>     // Todos os campos readonly
Pick<T, K>      // Subconjunto de campos
Omit<T, K>      // Todos exceto os especificados
Record<K, V>    // Mapa de chaves K para valores V
ReturnType<F>   // Tipo de retorno de uma função
```

**Uso prático:**

```typescript
// DTO para criação (sem id, sem createdAt)
type CreatePatientDTO = Omit<Patient, "id" | "createdAt">;

// DTO para update (tudo opcional exceto id)
type UpdatePatientDTO = Partial<Patient> & { id: string };
```

### Type Narrowing

```typescript
// typeof guard
function process(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase(); // TS sabe que é string aqui
  }
  return value.toFixed(2); // TS sabe que é number aqui
}

// Discriminated union
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

function handle(result: Result<User>) {
  if (result.success) {
    console.log(result.data.name); // TS sabe que data existe
  } else {
    console.error(result.error); // TS sabe que error existe
  }
}
```

### Enums vs Union Types

```typescript
// Enum (evitar na maioria dos casos)
enum Direction { Up, Down, Left, Right }

// Union type (preferido - mais simples, tree-shakeable)
type Direction = "up" | "down" | "left" | "right";

// const assertion (quando precisa de objeto)
const DIRECTIONS = { Up: "up", Down: "down" } as const;
type Direction = typeof DIRECTIONS[keyof typeof DIRECTIONS];
```

## Quando usar

- **TypeScript:** qualquer projeto não-trivial, especialmente em equipe
- **`any`:** quase nunca. Se precisa, use `unknown` com type guard
- **`as` (type assertion):** quando você sabe mais que o compilador, mas com cuidado
- **Generics:** funções/classes que trabalham com múltiplos tipos

## Armadilhas comuns

- **`any` everywhere:** derrota o propósito do TypeScript. Configurar `strict: true` no tsconfig.
- **Type assertions (`as`):** contornam o type checker. Usar type narrowing quando possível.
- **Enum pitfalls:** enums numéricos permitem valores inválidos. Preferir union types.
- **Over-typing:** tipar cada variável local é desnecessário. TypeScript infere bem com `const` e `let`.
- **Confundir runtime e compile-time:** tipos não existem em runtime. Não tentar fazer `typeof` em interfaces.

## Na prática (da minha experiência)

> Em projetos React com TypeScript, defino interfaces para props de componentes e types para estados complexos. No backend Node.js, uso generics para criar repositórios tipados — um `Repository<Patient>` tem os métodos certos com os tipos certos automaticamente. A migração gradual de JavaScript para TypeScript é possível com `allowJs: true` no tsconfig, o que permite converter arquivo por arquivo.

## How to explain in English

"TypeScript is essential for any production JavaScript project, in my opinion. Coming from Java, I appreciate having compile-time type safety — it catches entire categories of bugs before the code even runs.

I leverage TypeScript's type system extensively. I use discriminated unions for API responses — a `Result<T>` type that's either `{ success: true, data: T }` or `{ success: false, error: string }`. The compiler forces you to check the `success` field before accessing `data`, eliminating null reference errors.

Generics are where TypeScript really shines. I create generic repository interfaces, generic API response wrappers, and generic React hooks that maintain type safety throughout the application. The utility types like `Partial<T>`, `Pick<T, K>`, and `Omit<T, K>` are incredibly useful for creating derived types without duplication.

One thing I've learned is to let TypeScript infer types when it can. Over-annotating every variable makes the code noisy without adding safety. I annotate function parameters and return types, but let inference handle local variables."

### Key vocabulary

- tipagem estática → static typing
- inferência de tipo → type inference
- tipo genérico → generic type
- tipo union → union type: `A | B`
- tipo intersection → intersection type: `A & B`
- estreitamento de tipo → type narrowing
- asserção de tipo → type assertion (`as`)
- guarda de tipo → type guard

## Recursos

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/) — documentação oficial
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/) — livro gratuito
- [Total TypeScript](https://www.totaltypescript.com/) — curso avançado

## Veja também

- [[JavaScript Fundamentals]]
- [[Node.js]]
- [[React]]
