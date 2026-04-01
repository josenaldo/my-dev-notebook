---
title: "JavaScript Fundamentals"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - javascript
  - entrevista
publish: false
---

# JavaScript Fundamentals

Os conceitos core de JavaScript que diferenciam um senior — do event loop ao prototype chain.

## O que é

JavaScript é a linguagem da web. Single-threaded, com tipagem dinâmica, baseada em protótipos, e com um modelo de concorrência baseado em event loop. Para entrevistas, o foco é em: closures, event loop, promises/async-await, prototypes, e ES6+ features.

## Como funciona

### Event Loop

O coração da concorrência em JavaScript. Single-threaded, mas não-blocking.

```text
Call Stack → executa código síncrono
    ↓ (quando vazio)
Microtask Queue → Promises, queueMicrotask (prioridade)
    ↓ (quando vazia)
Macrotask Queue → setTimeout, setInterval, I/O callbacks
```

**Implicação:** código síncrono sempre executa antes de callbacks assíncronos. Um loop infinito síncrono bloqueia tudo.

### Closures

Uma função que "lembra" do escopo onde foi criada, mesmo após esse escopo encerrar.

```javascript
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    getCount: () => count
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
```

**Use cases:** encapsulamento, factory functions, memoization, módulos.

### Promises e Async/Await

```javascript
// Promise chain
fetchUser(id)
  .then(user => fetchOrders(user.id))
  .then(orders => processOrders(orders))
  .catch(err => handleError(err));

// Async/Await (preferido)
async function getUserOrders(id) {
  try {
    const user = await fetchUser(id);
    const orders = await fetchOrders(user.id);
    return processOrders(orders);
  } catch (err) {
    handleError(err);
  }
}
```

- **`Promise.all()`:** executa em paralelo, falha se qualquer uma falha
- **`Promise.allSettled()`:** executa em paralelo, retorna resultado de todas (sucesso ou falha)
- **`Promise.race()`:** retorna o primeiro a resolver/rejeitar

### Prototypes e `this`

- **Prototype chain:** herança baseada em objetos, não classes. `Object.create()`, `__proto__`.
- **`class` é syntax sugar:** por baixo, continua sendo protótipos.
- **`this`:** depende de como a função é chamada, não onde é definida.
  - Método de objeto: `this` = objeto
  - Arrow function: `this` = escopo léxico (de onde foi definida)
  - `call/apply/bind`: define `this` explicitamente

### Tipos e Coerção

- **Tipos primitivos:** `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`
- **`==` vs `===`:** `==` faz coerção de tipo, `===` não. **Sempre use `===`**.
- **Truthy/Falsy:** `0`, `""`, `null`, `undefined`, `NaN`, `false` são falsy. Todo o resto é truthy.
- **Optional chaining:** `user?.address?.city` — retorna `undefined` se qualquer parte for nula.
- **Nullish coalescing:** `value ?? 'default'` — usa default só se `null` ou `undefined` (não `0` ou `""`).

### ES6+ Features essenciais

- **Destructuring:** `const { name, age } = user;`
- **Spread/Rest:** `{ ...obj, newProp: value }`, `function(...args) {}`
- **Template literals:** `` `Hello ${name}` ``
- **Arrow functions:** `const fn = (x) => x * 2`
- **Modules:** `import/export` (ESM), `require` (CJS)
- **Map/Set:** coleções nativas (melhor que objetos para chaves não-string)
- **WeakMap/WeakRef:** referências fracas para evitar memory leaks

## Quando usar

- **JavaScript puro:** scripts simples, bibliotecas universais, prototipagem rápida
- **Com TypeScript:** qualquer projeto não-trivial (ver [[TypeScript]])
- **Arrow functions:** callbacks, funções curtas. Não para métodos de objeto (perde `this`).
- **Map vs Object:** Map quando chaves não são strings ou quando o número de entries muda muito.

## Armadilhas comuns

- **`this` em callbacks:** `setTimeout(obj.method, 100)` perde o `this`. Usar arrow function ou `.bind()`.
- **Floating point:** `0.1 + 0.2 !== 0.3`. Usar `Math.abs(a - b) < Number.EPSILON` ou bibliotecas para dinheiro.
- **`typeof null === 'object'`:** bug histórico do JavaScript. Checar null explicitamente.
- **`for...in` em arrays:** itera sobre todas as propriedades enumeráveis, não só índices. Usar `for...of` ou `.forEach()`.
- **Callback hell:** aninhar callbacks profundamente. Resolver com async/await.

## Na prática (da minha experiência)

> Em projetos fullstack, uso JavaScript/TypeScript tanto no frontend (React) quanto no backend (Node.js). A vantagem de usar a mesma linguagem nos dois lados é compartilhar types, validações e utilities. O event loop non-blocking é perfeito para APIs I/O-bound — um único processo Node.js pode servir milhares de conexões simultâneas sem precisar de thread pool complexo como em Java.

## How to explain in English

"JavaScript's single-threaded event loop model is what makes it unique. While Java handles concurrency with threads, JavaScript uses an event loop with a call stack and callback queues. This means it's non-blocking by nature — when you make an I/O call like a database query, the runtime continues processing other events while waiting for the response.

I use async/await extensively because it makes asynchronous code read like synchronous code. For parallel operations, `Promise.all()` is my go-to — for example, fetching a user's profile and their recent orders simultaneously instead of sequentially.

Closures are something I use daily without always thinking about it. Every React hook is a closure. Every factory function creates closures. Understanding closures deeply helps me reason about memory, scope, and encapsulation in JavaScript.

One area where my Java background helps is with type safety. In JavaScript, dynamic typing can lead to subtle bugs that are hard to catch. That's why I always use TypeScript in production code — it brings the compile-time safety I'm used to from Java."

### Key vocabulary

- laço de eventos → event loop
- encerramento léxico → closure
- promessa → promise: representação de valor futuro
- coerção de tipo → type coercion
- cadeia de protótipos → prototype chain
- desestruturação → destructuring
- operador de espalhamento → spread operator
- encadeamento opcional → optional chaining

## Recursos

- [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript) — referência completa
- [javascript.info](https://javascript.info/) — tutorial moderno
- [[Trilha JS-TS]] — trilha de aprendizado

## Veja também

- [[TypeScript]]
- [[Node.js]]
- [[React]]
