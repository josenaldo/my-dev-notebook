---
title: "Plano — Execução da Trilha TypeScript com React"
date: 2026-04-26
status: ready
type: plan
publish: false
---

# Trilha TypeScript com React — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produzir 15 notas atômicas + 1 MOC em `JavaScript/Frontend/TypeScript com React/`, em PT-BR, todas `publish: true`, cobrindo a intersecção idiomática TypeScript+React em 2026 — do mental model aos patterns type-level avançados — para um dev que já conhece JS+TS+React em nível médio e quer dominar o que separa senior de pleno.

**Architecture:** Trilha sequencial em 4 blocos (Mental model → Idiomas práticos → Type-level avançado → Fechamento) + 1 MOC central com 4 rotas alternativas. Cada nota é atômica e linkável, segue estrutura híbrida (TL;DR + corpo técnico), com code samples em TypeScript moderno (React 19, TS 5.x), wikilinks densos para [[TypeScript]] e [[React]] como referências paralelas, e seção "Em entrevista" para preparação internacional.

**Tech Stack:** Markdown + Obsidian Flavored Markdown (wikilinks, callouts, dataview), Quartz para publicação no site público. Sem código a executar — task é de pesquisa, escrita e validação dos exemplos TypeScript em [TypeScript Playground](https://www.typescriptlang.org/play) quando didático.

---

## ⚠️ Restrição absoluta — fabricação

A memória [Nunca inventar dados sobre o usuário](/home/josenaldo/.claude/projects/-home-josenaldo-repos-personal-codex-technomanticus/memory/feedback_no_fabrication.md) é regra inegociável neste plano. **Nenhuma nota pode atribuir ao autor experiências profissionais, projetos, clientes, métricas ou casos não-vividos.** A seção "Na prática" de cada nota usa:

- "Padrão observado em libs do ecossistema (MUI, Radix, Mantine)"
- "Caso típico em design systems"
- "Armadilha comum reportada na comunidade"
- Citações com fonte verificável (docs oficiais, posts identificados, repos públicos)
- Hipotéticos explícitos ("Imagine um componente de tabela genérico...")

Quando faltar contexto factual, **PERGUNTAR antes de escrever** — nunca preencher com plausibilidade.

---

## File Structure

16 arquivos em `JavaScript/Frontend/TypeScript com React/`:

```
JavaScript/Frontend/TypeScript com React/
├── TypeScript com React.md                                 # MOC (Task 1)
├── 01 - A tripla inferência - props, state, hooks.md       # Task 2
├── 02 - Inferir vs anotar - quando deixar o TS trabalhar.md # Task 3
├── 03 - Por que React.FC saiu de moda.md                   # Task 4
├── 04 - interface vs type vs satisfies para props.md       # Task 5
├── 05 - Tipando state e refs.md                            # Task 6
├── 06 - Tipando event handlers.md                          # Task 7
├── 07 - Tipando hooks customizados.md                      # Task 8
├── 08 - Tipando Context API.md                             # Task 9
├── 09 - Tipando reducers e state machines.md               # Task 10
├── 10 - Tipando formulários.md                             # Task 11
├── 11 - Tipando data fetching.md                           # Task 12
├── 12 - Generic components.md                              # Task 13
├── 13 - Polymorphic components com as prop.md              # Task 14
├── 14 - Compound components, slots, render props.md        # Task 15
└── 15 - Armadilhas, tsconfig, ferramentas.md               # Task 16
```

**Final integration (Task 17):** revisão de wikilinks cruzados + atualização de `Aprendizado/Trilha Frontend.md` e da seção "Veja também" de `JavaScript/Core/TypeScript.md` e `JavaScript/Frontend/React.md` para incluir entrada para a nova trilha.

---

## Template padrão (definido uma vez, aplicado a todas as 15 notas)

```markdown
---
title: "<título sem prefixo numérico>"
created: 2026-04-26
updated: 2026-04-26
type: concept
status: seedling
publish: true
tags:
  - typescript
  - react
  - typescript-react
  - frontend
  - <tags específicas: hooks, generics, forms, etc>
aliases:
  - <opcional: alternativas naturais de busca>
---

# <Título>

> [!abstract] TL;DR
> <2-4 linhas em PT-BR direto. Define o conceito-chave + a regra prática + por que importa.>

## O que é

<1-3 parágrafos definindo o conceito da nota. Pressupõe leitura de [[TypeScript]] e [[React]] ou consulta paralela.>

## Por que importa

<O problema que isso resolve no dia a dia. Onde o desenvolvedor "trava" sem esse conhecimento. Casos de uso reais.>

## Como funciona

<Aprofundamento técnico com code samples em TypeScript. Mostrar o pattern, depois variações. Edge cases.>

## Na prática

<Exemplo realista. Sem fabricar experiência do autor — usar "padrão observado em X", "lib Y faz isso", hipotéticos explícitos.>

## Armadilhas

<Gotchas específicos. O que erra normalmente, mensagens de erro confusas, anti-patterns comuns.>

## Em entrevista

<Frase pronta em inglês para explicar o conceito. Vocabulário-chave. Pergunta típica + resposta defensiva.>

## Veja também

- [[Outra nota da trilha]] — relação
- [[TypeScript]] — seção mãe
- [[React]] — seção mãe
```

### Variações permitidas

- **Nota 03 (`React.FC` saiu de moda):** seção "História" no lugar de "Como funciona"
- **Nota 15 (Armadilhas, tsconfig, ferramentas):** estrutura tópica (lista de armadilhas + bloco tsconfig + bloco ferramentas), não narrativa
- **MOC:** estrutura própria (abertura + Comece por aqui + Rotas alternativas + dataview de Todas as notas), `type: moc`

### Critérios de qualidade (rubrica aplicada por nota)

- [ ] TL;DR existe e é compreensível em <30 segundos
- [ ] 2+ wikilinks para outras notas da trilha + 1+ wikilink para [[TypeScript]] ou [[React]]
- [ ] 3+ code samples em TypeScript moderno (com comentários em PT-BR onde didático)
- [ ] Seção "Em entrevista" presente, com pelo menos 1 frase pronta em inglês
- [ ] Seção "Armadilhas" presente, com pelo menos 2 armadilhas concretas
- [ ] Frontmatter completo (`publish: true`, `status: seedling`, tags básicas + específicas)
- [ ] Versões assumidas declaradas se relevante (React 19, TS 5.x)
- [ ] PT-BR natural; termos técnicos em inglês mantidos (props, generics, refs, etc)
- [ ] **Zero atribuição de experiência pessoal ao autor** (regra absoluta)
- [ ] Nenhuma alegação técnica não-trivial sem fonte ou code sample que comprove

---

## Bibliografia centralizada (consulta rápida durante a escrita)

### Fontes-âncora (apontadas pelo autor)

- **Frontend Joy:** `https://www.frontendjoy.com/p/typescript-to-know-for-react`
- **Microsoft conversion guide:** `https://github.com/Microsoft/TypeScript-React-Conversion-Guide`

### Referências da comunidade (já validadas)

- **React TypeScript Cheatsheet:** `https://react-typescript-cheatsheet.netlify.app/`
- **Total TypeScript (Matt Pocock):** `https://www.totaltypescript.com/` — módulo "Advanced React with TypeScript"
- **TypeScript Handbook — JSX:** `https://www.typescriptlang.org/docs/handbook/jsx.html`
- **React docs (referência canônica):** `https://react.dev/reference/react`
- **react-hook-form TS guide:** `https://react-hook-form.com/get-started#TypeScript`
- **TanStack Query TS guide:** `https://tanstack.com/query/latest/docs/framework/react/typescript`

### Notas mãe no vault

- `JavaScript/Core/TypeScript.md` — referência paralela; cada nota linka para seções relevantes
- `JavaScript/Frontend/React.md` — referência paralela; cada nota linka para seções relevantes
- `JavaScript/Core/JavaScript Fundamentals.md` — pré-requisito, raramente referenciado
- `JavaScript/Core/Testes em JavaScript.md` — onde testes tipados moram

### A buscar conforme necessidade (não pré-validar)

- Material canônico sobre polymorphic components (Total TypeScript tem; comunidade tem várias implementações)
- Discussão histórica sobre remoção de `React.FC` em starters populares (CRA, Next.js)
- React Compiler docs (`https://react.dev/learn/react-compiler`)
- ts-reset (`https://github.com/total-typescript/ts-reset`)

---

## Task 0: Pré-flight

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/` (diretório)

- [ ] **Step 1: Criar o diretório**

```bash
mkdir -p "JavaScript/Frontend/TypeScript com React"
```

- [ ] **Step 2: Confirmar memória de no-fabrication carregada**

Verificar que `MEMORY.md` lista `feedback_no_fabrication.md`. Se não estiver, abortar e pedir reload.

- [ ] **Step 3: Sanity check das fontes-âncora**

Disparar WebFetch em paralelo para as 2 fontes apontadas pelo autor:

```
WebFetch: https://www.frontendjoy.com/p/typescript-to-know-for-react
WebFetch: https://github.com/Microsoft/TypeScript-React-Conversion-Guide
```

Confirmar que estão acessíveis e capturar o conteúdo essencial. Microsoft conversion guide é antigo (apontado para conversão JS→TS); usar como fonte de patterns clássicos, não como referência 2026.

- [ ] **Step 4: Sanity check da idade do guia da Microsoft**

Verificar último commit do `Microsoft/TypeScript-React-Conversion-Guide` no GitHub. Se for >2 anos, mencionar isso explicitamente nas notas que o citarem (ex: "guia clássico da Microsoft de 2018, ainda útil para conceitos básicos mas não cobre Hooks").

---

## Wave 1 — Esqueleto (Tasks 1-2)

Definem vocabulário e estrutura. **Bloqueante** para Waves seguintes.

### Task 1: MOC central — `TypeScript com React.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/TypeScript com React.md`

**Sources:**
- Spec da trilha: `docs/superpowers/specs/2026-04-26-typescript-react-design.md` (seções 5 e 7)
- MOC de referência: `IA/Memória de Agentes/Memória de Agentes.md`

- [ ] **Step 1: Frontmatter**

```yaml
---
title: "TypeScript com React"
type: moc
publish: true
tags: [typescript, react, typescript-react, frontend, moc]
created: 2026-04-26
updated: 2026-04-26
---
```

- [ ] **Step 2: Abertura (3-4 frases)**

Conteúdo:
- Frase 1: o que esta trilha cobre (a intersecção idiomática TS+React, não TS isolado nem React isolado)
- Frase 2: por que existe — em 2026 TS é default, mas a intersecção tem patterns não-óbvios que separam senior de pleno
- Frase 3: pressuposto — leitor já leu (ou pode consultar) [[TypeScript]] e [[React]]; trilha é complementar, não substitui as mães
- Frase 4: o que entrega — do mental model ao type-level avançado, com seção "Em entrevista" para preparação internacional

- [ ] **Step 3: Callout de orientação**

```markdown
> [!info] Como ler
> A trilha pressupõe que você consultará [[TypeScript]] e [[React]] como referências paralelas. Cada nota linka explicitamente para a seção mãe quando precisa de um conceito de TS ou React que não é o foco. Para uma primeira leitura, siga a sequência 01 → 15. Para reforço pontual ou preparação de entrevista, use as rotas alternativas abaixo.
```

- [ ] **Step 4: Seção "Comece por aqui" (sequencial)**

Lista linear das 15 notas (01 → 15) com 1 linha de descrição cada. Exemplo:

```markdown
- [[01 - A tripla inferência - props, state, hooks|01 - A tripla inferência]] — como o TS pensa em React: props declaradas, state inferido, hooks tipados pela lib
- [[02 - Inferir vs anotar - quando deixar o TS trabalhar|02 - Inferir vs anotar]] — a regra prática: anote inputs públicos, deixe inferir o resto
...
- [[15 - Armadilhas, tsconfig, ferramentas]] — checklist final + tsconfig React+Vite/Next + ESLint + ts-reset
```

- [ ] **Step 5: Seção "Rotas alternativas"**

```markdown
## Rotas alternativas

### Rota entrevista (preparar perguntas frequentes em entrevistas internacionais)
[[03 - Por que React.FC saiu de moda]] → [[04 - interface vs type vs satisfies para props|04 - interface vs type vs satisfies]] → [[07 - Tipando hooks customizados]] → [[09 - Tipando reducers e state machines]] → [[12 - Generic components]] → [[13 - Polymorphic components com as prop|13 - Polymorphic components]]

### Rota produção (escrever bem no dia a dia, sem type-level avançado)
[[01 - A tripla inferência - props, state, hooks|01 - A tripla inferência]] → [[02 - Inferir vs anotar - quando deixar o TS trabalhar|02 - Inferir vs anotar]] → [[05 - Tipando state e refs]] → [[06 - Tipando event handlers]] → [[08 - Tipando Context API]] → [[10 - Tipando formulários]] → [[11 - Tipando data fetching]] → [[15 - Armadilhas, tsconfig, ferramentas]]

### Rota library author (escrever componentes reutilizáveis)
[[04 - interface vs type vs satisfies para props|04 - interface vs type vs satisfies]] → [[07 - Tipando hooks customizados]] → [[12 - Generic components]] → [[13 - Polymorphic components com as prop|13 - Polymorphic components]] → [[14 - Compound components, slots, render props|14 - Compound components]]

### Rota completa
Sequencial 01 → 15. Recomendada na primeira leitura.
```

- [ ] **Step 6: Seção "Todas as notas" com dataview**

````markdown
## Todas as notas

```dataview
LIST file.frontmatter.title
FROM "JavaScript/Frontend/TypeScript com React"
WHERE type != "moc"
SORT file.name ASC
```
````

- [ ] **Step 7: Seção "Veja também"**

```markdown
## Veja também

- [[TypeScript]] — deep dive da linguagem (referência paralela)
- [[React]] — deep dive da biblioteca (referência paralela)
- [[JavaScript Fundamentals]] — base
- [[Testes em JavaScript]] — para testes de componentes tipados
- [[Trilha Frontend]] — trilha de aprendizado mais ampla
```

- [ ] **Step 8: Quality check**

Verificar rubrica. MOC é exceção em "3+ code samples" (MOC não tem código).

- [ ] **Step 9: Commit**

```bash
git add "JavaScript/Frontend/TypeScript com React/TypeScript com React.md"
git commit -m "feat(typescript-react): MOC central da trilha"
```

---

### Task 2: Nota 01 — `01 - A tripla inferência - props, state, hooks.md`

**Por que é Wave 1:** define o vocabulário central que toda a trilha usa (props vs state vs hooks como três fontes de tipo distintas; JSX como `React.createElement`; `JSX.IntrinsicElements`).

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/01 - A tripla inferência - props, state, hooks.md`

**Sources:**
- TypeScript Handbook — JSX: `https://www.typescriptlang.org/docs/handbook/jsx.html`
- React docs — Components: `https://react.dev/learn/your-first-component`
- React docs — Hooks reference: `https://react.dev/reference/react/hooks`
- [[TypeScript]] (seções "Generics", "TypeScript em frontend (React)")
- [[React]] (seções "Componentes e JSX", "Hooks essenciais")

- [ ] **Step 1: Pré-research**

Confirmar que `JSX.IntrinsicElements` ainda é a interface usada (vs `React.JSX.IntrinsicElements` em React 19). WebFetch da seção JSX do Handbook.

- [ ] **Step 2: Aplicar template — frontmatter**

```yaml
---
title: "A tripla inferência - props, state, hooks"
created: 2026-04-26
updated: 2026-04-26
type: concept
status: seedling
publish: true
tags: [typescript, react, typescript-react, frontend, mental-model, jsx, inference]
aliases:
  - Tripla inferência
  - JSX e tipos
---
```

- [ ] **Step 3: TL;DR**

Em React+TS, há três fontes distintas de inferência de tipos: (1) props — declaradas pelo dev na assinatura do componente; (2) state — inferido do initializer ou anotado quando inicializado com `null`/`[]`/`{}`; (3) hooks — return types vêm da lib (React, TanStack Query, etc). Entender as três como sistemas separados elimina a maior parte dos erros confusos do iniciante.

- [ ] **Step 4: Seção "O que é"**

Conteúdo:
- JSX é apenas açúcar para `React.createElement(type, props, ...children)`. O TS sabe disso e checa com base nas tipagens da lib.
- `JSX.IntrinsicElements` é a interface que define todos os elementos HTML/SVG nativos com seus props. `<button onClick={...}>` valida contra `JSX.IntrinsicElements['button']`.
- Componentes customizados expõem suas props via assinatura da função. Não há mágica — o TS lê a assinatura.
- Hooks têm tipos vindos das libs. `useState` tem inferência elaborada; `useRef` tem 3 overloads que confundem.

- [ ] **Step 5: Seção "Por que importa"**

Mensagens de erro como `Type '{ children: Element[] }' is missing the following properties...` ficam triviais quando você sabe exatamente *de onde* o TS está tentando inferir o tipo. Sem o mental model, o dev "luta com o compilador". Com ele, o dev "lê" o que o compilador está dizendo.

- [ ] **Step 6: Seção "Como funciona" — code samples**

Sample 1 — props inferidas da assinatura:

```typescript
type ButtonProps = {
  variant: 'primary' | 'secondary';
  onClick: () => void;
  children: React.ReactNode;
};

function Button({ variant, onClick, children }: ButtonProps) {
  return <button onClick={onClick} className={variant}>{children}</button>;
}

// TS rejeita props errados:
// <Button variant="danger" onClick={() => {}}>X</Button>
//                ^^^^^^^^ Type '"danger"' is not assignable...
```

Sample 2 — state inferido vs anotado:

```typescript
const [count, setCount] = useState(0);          // count: number (inferido)
const [user, setUser] = useState<User | null>(null);  // anotado — null sozinho não dá pra inferir
const [items, setItems] = useState<Item[]>([]); // anotado — array vazio não dá pra inferir
```

Sample 3 — hook return types vindos da lib:

```typescript
import { useQuery } from '@tanstack/react-query';

const { data, error, isLoading } = useQuery({
  queryKey: ['user', id],
  queryFn: () => fetchUser(id),
});
// data: User | undefined  (inferido pelo return de queryFn)
// error: Error | null      (default da TanStack)
// isLoading: boolean       (sempre)
```

Sample 4 — JSX intrinsic via `JSX.IntrinsicElements`:

```typescript
type ButtonNativeProps = JSX.IntrinsicElements['button'];
// ButtonNativeProps tem todas as props nativas do <button>: onClick, disabled, type, etc.

type MyButtonProps = ButtonNativeProps & { variant: 'primary' | 'secondary' };
// agora MyButton aceita TUDO que <button> aceita + variant
```

- [ ] **Step 7: Seção "Na prática"**

Padrão observado em libs do ecossistema (Radix, Mantine, MUI): props customizadas estendem `ComponentPropsWithoutRef<'button'>` (ou similar) para herdar todos os atributos HTML nativos. A nota 13 aprofunda esse pattern em polymorphic components.

- [ ] **Step 8: Seção "Armadilhas"**

- `useState(null)` infere `null` literal — não `null | T`. Sempre anote.
- `useRef<HTMLDivElement>(null)` retorna `RefObject<HTMLDivElement>` (`{ current: HTMLDivElement | null }`); `useRef<number>(0)` retorna `MutableRefObject<number>`. Nota 05 cobre a diferença.
- `<Component<T> />` em arquivo `.tsx` precisa de vírgula extra: `<Component<T,> />` ou `<Component<T extends unknown> />`. JSX confunde com generic. Nota 12 aprofunda.

- [ ] **Step 9: Seção "Em entrevista"**

> "In React with TypeScript, there are three sources of inference. Props come from the component's signature — I declare them and TS validates the JSX against them. State is inferred from `useState`'s initial value when it's a primitive, but I have to annotate when starting with `null` or empty arrays. Hooks return types come from the library — `useQuery` from TanStack, for example, infers the data type from my `queryFn` return. Knowing these three sources separately makes confusing error messages readable."

Vocabulário-chave: *intrinsic elements*, *type inference*, *initial value*, *return type*.

- [ ] **Step 10: Seção "Veja também"**

```markdown
- [[02 - Inferir vs anotar - quando deixar o TS trabalhar]]
- [[05 - Tipando state e refs]]
- [[12 - Generic components]]
- [[TypeScript]] — seção "Generics" e "TypeScript em frontend (React)"
- [[React]] — seção "Componentes e JSX"
```

- [ ] **Step 11: Quality check**

Rubrica completa. Code samples ≥ 3 ✓ (4 samples). Wikilinks ≥ 2 ✓.

- [ ] **Step 12: Commit**

```bash
git add "JavaScript/Frontend/TypeScript com React/01 - A tripla inferência - props, state, hooks.md"
git commit -m "feat(typescript-react): nota 01 - A tripla inferência"
```

---

## Wave 2 — Mental model (Tasks 3-5)

Completam o bloco 1. Cada task pode ser executada em paralelo (sem dependência entre si) após a Task 2.

### Task 3: Nota 02 — `02 - Inferir vs anotar - quando deixar o TS trabalhar.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/02 - Inferir vs anotar - quando deixar o TS trabalhar.md`

**Sources:**
- Total TypeScript — "When to annotate" (módulo Beginner's TypeScript)
- TypeScript Handbook — Type Inference: `https://www.typescriptlang.org/docs/handbook/type-inference.html`
- TypeScript 4.9 release notes (`satisfies`)
- [[TypeScript]] (seção "Tipos básicos", "Generics")

- [ ] **Step 1: Pré-research**

Refresh sobre `satisfies` (TS 4.9+) e seu uso idiomático em props/objetos React.

- [ ] **Step 2: Aplicar template — frontmatter**

```yaml
---
title: "Inferir vs anotar - quando deixar o TS trabalhar"
created: 2026-04-26
updated: 2026-04-26
type: concept
status: seedling
publish: true
tags: [typescript, react, typescript-react, frontend, mental-model, inference, satisfies]
aliases:
  - Quando anotar tipos em React
  - satisfies operator
---
```

- [ ] **Step 3: TL;DR**

A regra: anote inputs públicos (props, parâmetros, return types de funções exportadas), deixe inferir o resto. Quando inferência falha (initializers `null`, `[]`, `{}`), anote explicitamente. Use `satisfies` quando quiser validar shape sem perder literal types.

- [ ] **Step 4: Seção "O que é"**

Inferência: TS deduz o tipo a partir do valor. `const x = 5` → `x: number`. Funciona muito bem em JS comum, e particularmente bem em React (`useState(0)` → `number`).
Anotação: `const x: User = ...`. Necessária quando o tipo não pode ser deduzido do valor (`useState<User | null>(null)`) ou quando você quer ser explícito como contrato.
`satisfies` (TS 4.9+): valida que um valor satisfaz um tipo, sem widening. Útil para configs, constantes, props parciais.

- [ ] **Step 5: Seção "Por que importa"**

Excesso de anotação polui o código e frequentemente *piora* a inferência (anotar `const items: string[] = ['a', 'b']` perde o literal `['a', 'b']`). Falta de anotação em fronteiras públicas (props de componentes exportados, return de hooks customizados) faz o tipo "vazar" e ficar implicit. A regra prática evita os dois extremos.

- [ ] **Step 6: Seção "Como funciona" — code samples**

Sample 1 — quando inferir é melhor:

```typescript
// Inferido
const [count, setCount] = useState(0);  // number — claro
const items = ['a', 'b', 'c'];          // string[] — claro
const user = { name: 'Maria', age: 30 }; // { name: string; age: number } — claro

// Anotação aqui é poluição:
// const [count, setCount]: [number, ...] = useState<number>(0);  // ruim
```

Sample 2 — quando anotar é necessário:

```typescript
// useState com null
const [user, setUser] = useState<User | null>(null);

// Array vazio
const [items, setItems] = useState<Item[]>([]);

// Objeto vazio
const [form, setForm] = useState<Partial<FormData>>({});

// Map/Set (não há literal type útil)
const [cache, setCache] = useState<Map<string, User>>(new Map());
```

Sample 3 — anotação em fronteira pública (props):

```typescript
// Sem anotação — TS infere {}, perdendo o contrato
function Button({ onClick, children }) {  // ERRO em strict mode (implicit any)
  return <button onClick={onClick}>{children}</button>;
}

// Com anotação — contrato claro
type ButtonProps = { onClick: () => void; children: React.ReactNode };

function Button({ onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}
```

Sample 4 — `satisfies` para validar sem perder literal:

```typescript
// Sem satisfies — perde literal type
const ROUTES = {
  home: '/',
  user: '/users/:id',
} as const;
// ROUTES.home: '/' (literal preservado)

// Com satisfies — valida shape mas mantém literal
type RouteMap = Record<string, string>;
const ROUTES = {
  home: '/',
  user: '/users/:id',
} satisfies RouteMap;
// ROUTES.home: '/' (literal preservado E shape validado)
```

Sample 5 — return de hook customizado: anotar ou inferir?

```typescript
// Inferido — o tipo "vaza" da implementação
function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  return [on, () => setOn(s => !s)] as const;
}
// Tipo: readonly [boolean, () => void]

// Anotado — explícito como contrato
function useToggle(initial = false): readonly [boolean, () => void] {
  const [on, setOn] = useState(initial);
  return [on, () => setOn(s => !s)] as const;
}
// Idêntico em uso, mas API explícita
```

- [ ] **Step 7: Seção "Na prática"**

Padrão observado em libs sérias (TanStack, Radix, MUI): tipos exportados são sempre anotados; tipos internos (variáveis locais, intermediários) deixam inferir. O critério é "isso atravessa um boundary?" — se sim, anotar. Se não, deixar inferir.

- [ ] **Step 8: Seção "Armadilhas"**

- Anotar variáveis locais com tipos primitivos: `const x: number = 5` é poluição.
- Não anotar return de hook customizado: o tipo passa a ser "o que a implementação diz", não "o que o contrato promete".
- Confundir `as const` com `satisfies`: `as const` *altera* o tipo (mais estreito); `satisfies` apenas *valida* sem alterar.
- Esquecer de anotar em arrays/objects vazios: TS infere `never[]` ou `{}`, e qualquer push/atribuição depois falha.

- [ ] **Step 9: Seção "Em entrevista"**

> "My rule: annotate what crosses a boundary, infer the rest. Component props, exported function signatures, custom hook return types — those need explicit annotations because they're contracts other code depends on. Local variables, intermediate values, primitives — those should infer; annotating them is just noise. The exception is when inference fails: `useState(null)` infers `null` literally, `useState([])` infers `never[]`, so I annotate. For configs and constants where I want to validate shape but keep literal types, I use `satisfies`."

Vocabulário-chave: *type widening*, *literal types*, *contract*, *satisfies operator*.

- [ ] **Step 10: Seção "Veja também"**

```markdown
- [[01 - A tripla inferência - props, state, hooks]]
- [[04 - interface vs type vs satisfies para props]]
- [[07 - Tipando hooks customizados]]
- [[TypeScript]] — seção "Tipos básicos"
```

- [ ] **Step 11: Quality check + Commit**

```bash
git add "JavaScript/Frontend/TypeScript com React/02 - Inferir vs anotar - quando deixar o TS trabalhar.md"
git commit -m "feat(typescript-react): nota 02 - Inferir vs anotar"
```

---

### Task 4: Nota 03 — `03 - Por que React.FC saiu de moda.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/03 - Por que React.FC saiu de moda.md`

**Sources:**
- React TypeScript Cheatsheet — "Function Components": `https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/function_components`
- Discussão histórica em PRs do DefinitelyTyped (buscar: "DefinitelyTyped React.FC children")
- Posts identificados sobre o tema (buscar conforme necessidade)
- [[React]] (seções "Componentes e JSX")

- [ ] **Step 1: Pré-research**

WebFetch da página do React TypeScript Cheatsheet sobre Function Components. Buscar 1-2 posts da comunidade sobre o tema (ex: kentcdodds, Sébastien Lorber, Total TypeScript).

**Importante:** confirmar fatos históricos com fontes; não fabricar quem disse o quê.

- [ ] **Step 2: Aplicar template — frontmatter**

```yaml
---
title: "Por que React.FC saiu de moda"
created: 2026-04-26
updated: 2026-04-26
type: concept
status: seedling
publish: true
tags: [typescript, react, typescript-react, frontend, mental-model, components, history]
aliases:
  - React.FC vs annotation
  - FunctionComponent
---
```

- [ ] **Step 3: TL;DR**

`React.FC` (alias `React.FunctionComponent`) era o padrão para tipar componentes funcionais até ~2020-2021. A comunidade se afastou por três razões: (1) injetava `children` implicitamente em todos os componentes; (2) atrapalhava componentes genéricos; (3) padrões com `defaultProps` ficavam awkward. Hoje o idiomático é anotar props como parâmetro: `function Button({ ... }: Props)`.

- [ ] **Step 4: Seção "O que é"**

`React.FC<Props>` é um type alias que descreve a assinatura de um componente funcional. Historicamente, tipava o componente como `(props: Props & { children?: ReactNode }) => ReactElement`.

- [ ] **Step 5: Seção "História"** (variação do template)

- 2018-2020: `React.FC` era o default em starters (CRA, Next.js).
- 2020-2021: PRs em DefinitelyTyped removeram `children` implícito do `React.FC`, mas a comunidade já tinha começado a evitá-lo.
- 2021+: starters como Vite e novos templates usam anotação direta.
- 2023+: React TypeScript Cheatsheet removeu `React.FC` da recomendação principal.

(Confirmar datas via fonte primária no Step 1 antes de afirmar.)

- [ ] **Step 6: Seção "Por que importa" + code samples**

Sample 1 — `children` implícito (problema 1):

```typescript
// Antigamente (com React.FC)
const Heading: React.FC<{ title: string }> = ({ title, children }) => (
  <h1>{title}{children}</h1>  // children é inferido como ReactNode
);

// Heading deveria explicitar children no tipo, mas FC injetava silenciosamente.
// Resultado: <Heading title="X">qualquer coisa</Heading> compilava sempre,
// mesmo quando o componente não deveria aceitar children.
```

Sample 2 — generics ficam estranhos (problema 2):

```typescript
// Generic component com React.FC — não funciona
const List: React.FC<{ items: T[] }> = ({ items }) => (...);
//          ^ T não está em scope

// Workaround feio:
const List = <T,>({ items }: { items: T[] }): React.ReactElement => (...);
// Não é mais React.FC

// Idiomático moderno (sem React.FC):
function List<T>({ items }: { items: T[] }) {
  return <ul>{items.map(...)}</ul>;
}
```

Sample 3 — o que usar hoje:

```typescript
type ButtonProps = {
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;  // explícito quando aceitar
};

// Anotação direta — idiomática
function Button({ onClick, variant = 'primary', children }: ButtonProps) {
  return <button onClick={onClick} className={variant}>{children}</button>;
}

// Arrow function — também válido
const Button = ({ onClick, variant = 'primary', children }: ButtonProps) => (
  <button onClick={onClick} className={variant}>{children}</button>
);
```

- [ ] **Step 7: Seção "Quando ainda faz sentido"**

Casos remanescentes para `React.FC`:
- Equipes/codebases legados onde a consistência com código antigo importa mais que ergonomia
- Quando você quer marcar **explicitamente** "isto é um componente" para leitores

Em ambos os casos, é preferência de estilo. Funcionalmente, anotação direta cobre tudo.

- [ ] **Step 8: Seção "Armadilhas"**

- Trocar `React.FC` por anotação direta sem **adicionar `children` no tipo** quando o componente realmente aceita filhos.
- Confundir `React.FC` com `React.FunctionComponent` (são aliases, nada muda).
- Em React 18+, `React.FC` *sem* `children` implícito é o default — ainda assim, anotação direta é preferida pela comunidade.

- [ ] **Step 9: Seção "Em entrevista"**

> "`React.FC` was the standard until around 2020. The community moved away from it for three reasons. First, it injected `children` implicitly into every component, even ones that shouldn't accept them. Second, generic components became awkward — you can't easily express `React.FC<Props<T>>` without weird syntax. Third, `defaultProps` patterns didn't compose well. Today the idiom is annotating props as the parameter: `function Button({ onClick, children }: Props)`. It's simpler, plays well with generics, and forces you to be explicit about whether children is part of the API."

Vocabulário-chave: *function component*, *children prop*, *implicit*, *idiomatic*.

- [ ] **Step 10: Seção "Veja também"**

```markdown
- [[01 - A tripla inferência - props, state, hooks]]
- [[04 - interface vs type vs satisfies para props]]
- [[12 - Generic components]]
- [[React]] — seção "Componentes e JSX"
```

- [ ] **Step 11: Quality check + Commit**

```bash
git add "JavaScript/Frontend/TypeScript com React/03 - Por que React.FC saiu de moda.md"
git commit -m "feat(typescript-react): nota 03 - Por que React.FC saiu de moda"
```

---

### Task 5: Nota 04 — `04 - interface vs type vs satisfies para props.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/04 - interface vs type vs satisfies para props.md`

**Sources:**
- TypeScript Handbook — Interfaces vs Type Aliases
- React TypeScript Cheatsheet — "Types or Interfaces?"
- [[TypeScript]] (seção "Interfaces e Type aliases")

- [ ] **Step 1: Pré-research**

Confirmar convenção atual de libs do ecossistema React: MUI (interface), Mantine (interface), Radix (mistura), TanStack (type). Capturar exemplo de cada.

- [ ] **Step 2: Aplicar template — frontmatter**

```yaml
---
title: "interface vs type vs satisfies para props"
created: 2026-04-26
updated: 2026-04-26
type: concept
status: seedling
publish: true
tags: [typescript, react, typescript-react, frontend, mental-model, props, interface, type, satisfies]
aliases:
  - Interface vs type para props
  - satisfies em props React
---
```

- [ ] **Step 3: TL;DR**

Para props de componentes: `interface` quando você quer **declaration merging** (estender tipos de libs externas) ou está escrevendo uma lib que outros vão estender. `type` quando precisa de unions, intersections, mapped, conditional. `satisfies` para validar props parciais sem perder literal types. Convenção: sufixo `Props` (ex: `ButtonProps`).

- [ ] **Step 4: Seção "O que é"**

Resumo (refere a [[TypeScript]] para deep dive):
- `interface` define shape de objeto, suporta declaration merging, suporta `extends`.
- `type` é mais flexível — suporta unions, intersections, mapped, conditional, primitives renomeados.
- `satisfies` valida que um valor obedece um tipo sem widening.

- [ ] **Step 5: Seção "Por que importa"**

A escolha tem impacto em três cenários: (1) componentes que estendem props de elementos HTML; (2) componentes que aceitam props variantes (discriminated unions); (3) componentes em libs públicas. A escolha errada causa código mais verboso ou menos extensível.

- [ ] **Step 6: Seção "Como funciona" — code samples**

Sample 1 — interface para extension:

```typescript
// Quando quero permitir que outros estendam
interface ButtonProps extends React.ComponentPropsWithoutRef<'button'> {
  variant?: 'primary' | 'secondary';
}

// Outra lib pode estender:
interface FancyButtonProps extends ButtonProps {
  glow: boolean;
}
```

Sample 2 — type para union de variants:

```typescript
// Discriminated union — props mudam conforme variant
type ButtonProps =
  | { variant: 'icon'; icon: ReactNode; label: string }
  | { variant: 'text'; children: ReactNode }
  | { variant: 'split'; primary: ReactNode; secondary: ReactNode };

// TS força handler a verificar variant antes de acessar a prop específica
function Button(props: ButtonProps) {
  if (props.variant === 'icon') return <span>{props.icon}</span>;
  // props.children não existe aqui — narrowed
  ...
}
```

Sample 3 — satisfies para defaults parciais:

```typescript
const DEFAULT_BUTTON_PROPS = {
  variant: 'primary',
  size: 'md',
  disabled: false,
} satisfies Partial<ButtonProps>;

// satisfies valida shape sem widening:
// DEFAULT_BUTTON_PROPS.variant: 'primary' (literal preservado)
// vs
// const X: Partial<ButtonProps> = { variant: 'primary', ... };
// X.variant: ButtonProps['variant'] | undefined (widened)
```

Sample 4 — convenção de nomenclatura:

```typescript
// Sufixo Props — convenção do ecossistema
type ButtonProps = { ... };
type ModalProps = { ... };
type FormProps = { ... };

// Não use I-prefix (anti-pattern em TS):
// interface IButtonProps { ... }  // ruim — convenção C#/Java, não TS
```

- [ ] **Step 7: Seção "Na prática" — convenções de libs**

Padrão observado:
- **MUI / Mantine:** `interface` em todos os componentes públicos (permite users estenderem via `declaration merging` para customizar themes).
- **Radix UI:** mistura — `interface` quando estende HTML element, `type` quando é puro shape.
- **TanStack Query:** `type` em quase tudo (preferência por composability via union/intersection).

A regra prática: **`interface` quando outros vão estender; `type` quando o tipo é "fechado" do meu lado**. Em apps de produto (não libs), `type` é mais simples e cobre 95% dos casos.

- [ ] **Step 8: Seção "Armadilhas"**

- Misturar `interface` e `type` no mesmo codebase sem critério: cada arquivo decide. Resultado: code review confuso, refactor difícil. Escolha uma convenção e documente.
- Tentar union com `interface`: não funciona — interfaces não fazem `|`. Use `type`.
- Esquecer que `interface` permite **declaration merging acidental**: se duas interfaces com mesmo nome existirem no mesmo escopo, viram uma só. Pode causar bugs estranhos em codebases grandes.
- Usar `interface` por hábito quando precisa de mapped types. `type` é a única opção em mapped/conditional.

- [ ] **Step 9: Seção "Em entrevista"**

> "For component props, my rule is: `interface` when consumers might want to extend the type — declaration merging works on interfaces, not types. `type` when I need unions, intersections, mapped types, or conditional types — interfaces can't do those. In practice, in product code I default to `type` because it's simpler and covers most cases. In library code, I lean toward `interface` for public APIs because users can extend them. For partial defaults or configs, I use `satisfies` instead of `:` annotation, because `satisfies` validates shape without widening literal types."

Vocabulário-chave: *declaration merging*, *type alias*, *literal type widening*, *discriminated union*.

- [ ] **Step 10: Seção "Veja também"**

```markdown
- [[02 - Inferir vs anotar - quando deixar o TS trabalhar]]
- [[09 - Tipando reducers e state machines]] — discriminated unions em ação
- [[12 - Generic components]]
- [[TypeScript]] — seção "Interfaces e Type aliases"
```

- [ ] **Step 11: Quality check + Commit**

```bash
git add "JavaScript/Frontend/TypeScript com React/04 - interface vs type vs satisfies para props.md"
git commit -m "feat(typescript-react): nota 04 - interface vs type vs satisfies"
```

---

## Wave 3 — Idiomas práticos (Tasks 6-12)

7 notas. Podem ser paralelizadas após Wave 2.

### Task 6: Nota 05 — `05 - Tipando state e refs.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/05 - Tipando state e refs.md`

**Sources:**
- React docs — `useState`, `useRef`, `useImperativeHandle`
- React TypeScript Cheatsheet — Hooks
- [[React]] (seção "Hooks essenciais")

**Pattern of work (segue o mesmo formato das tasks anteriores; estrutura abreviada para o resto do plano para reduzir verbosidade):**

- [ ] Step 1: Pré-research — confirmar diferença `RefObject` vs `MutableRefObject` em React 19
- [ ] Step 2: Frontmatter (template padrão; tags adicionar `state, refs, useState, useRef`)
- [ ] Step 3: TL;DR — `useState` infere de primitivos mas precisa anotar com `null`/`[]`/`{}`. `useRef` tem 3 overloads: ref de DOM (passa `null`), ref mutable (passa valor), ref que pode ser `null` em outras situações.
- [ ] Step 4: "O que é" — duas categorias de ref: DOM ref (lida pelo React, passada para JSX) vs mutable ref (controle do dev, sobrevive entre renders sem causar re-render).
- [ ] Step 5: "Como funciona" — code samples (mín 4):
  - `useState<T>` quando initializer é `null`/`[]`/`{}`/`Map`/`Set`
  - `useRef<HTMLInputElement>(null)` para DOM, `useRef<number>(0)` para counter mutable
  - `RefObject<T>` (`{ current: T | null }`) vs `MutableRefObject<T>` (`{ current: T }`)
  - Callback ref: `const setRef = (node: HTMLInputElement | null) => { ... }`
  - `useImperativeHandle` tipado com `forwardRef`
- [ ] Step 6: "Na prática" — pattern de hook que retorna ref (`useFocusOnMount` → `RefObject<HTMLInputElement>`)
- [ ] Step 7: "Armadilhas" — `useRef<T>(null!)` é anti-pattern (force non-null); ref ainda pode ser `null` quando o componente está unmounting
- [ ] Step 8: "Em entrevista" — frase pronta sobre as 3 categorias de refs
- [ ] Step 9: "Veja também" — `[[01 - A tripla inferência]]`, `[[06 - Tipando event handlers]]`, `[[13 - Polymorphic components]]`, `[[React]]`
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 05 - Tipando state e refs`)

---

### Task 7: Nota 06 — `06 - Tipando event handlers.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/06 - Tipando event handlers.md`

**Sources:**
- React docs — Common (events): `https://react.dev/reference/react-dom/components/common`
- React TypeScript Cheatsheet — Events

- [ ] Step 1: Pré-research — confirmar tipos de eventos em React 19 (`SyntheticEvent` ainda existe? mudou?)
- [ ] Step 2: Frontmatter (tags: `events, handlers`)
- [ ] Step 3: TL;DR — eventos React são *synthetic* (wrappers em torno do DOM nativo). Os tipos vivem em `React.MouseEvent<T>`, `React.ChangeEvent<T>`, `React.FormEvent<T>`. O genérico `T` é o elemento HTML (`HTMLButtonElement`, `HTMLInputElement`, etc).
- [ ] Step 4: "O que é" — diferença `SyntheticEvent` (do React) vs `MouseEvent` (DOM nativo, em listeners imperativos via `addEventListener`).
- [ ] Step 5: "Como funciona" — code samples (mín 5):
  - `onClick: (e: React.MouseEvent<HTMLButtonElement>) => void`
  - `onChange: (e: React.ChangeEvent<HTMLInputElement>) => void` — `e.target.value: string`
  - `onSubmit: (e: React.FormEvent<HTMLFormElement>) => void` — `e.preventDefault()`
  - `currentTarget` vs `target`: por que importa (`currentTarget` é typed como o elemento que tem o listener; `target` pode ser child)
  - Handler reutilizável: `function handleChange<T>(e: React.ChangeEvent<HTMLInputElement>): T { ... }`
  - Listener nativo (em `useEffect` com `addEventListener`): `(e: MouseEvent) => void` — sem React.
- [ ] Step 6: "Na prática" — pattern de form handler único: `(e: React.ChangeEvent<HTMLInputElement>) => setForm(f => ({ ...f, [e.target.name]: e.target.value }))`
- [ ] Step 7: "Armadilhas" — `e.target.value` em `onClick` (target é qualquer node, não input); confundir `MouseEvent` (DOM) com `React.MouseEvent` (synthetic); usar `any` em handlers
- [ ] Step 8: "Em entrevista" — frase sobre synthetic vs nativo
- [ ] Step 9: "Veja também" — `[[05 - Tipando state e refs]]`, `[[10 - Tipando formulários]]`, `[[React]]`
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 06 - Tipando event handlers`)

---

### Task 8: Nota 07 — `07 - Tipando hooks customizados.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/07 - Tipando hooks customizados.md`

**Sources:**
- Total TypeScript — module Advanced React with TypeScript
- React TypeScript Cheatsheet — Custom Hooks

- [ ] Step 1: Pré-research — confirmar pattern de overload em hooks (TS supporta overloads em function declarations; `useStorage(key)` vs `useStorage(key, default)`)
- [ ] Step 2: Frontmatter (tags: `hooks, custom-hooks, generics, overloads`)
- [ ] Step 3: TL;DR — hooks customizados devem retornar tupla `as const` (não objeto, na maioria dos casos) para preservar nomes posicionais. Generics inferidos do parâmetro permitem `useFetch<User>('/api/me')`. Overloads cobrem assinaturas múltiplas. Discriminated union no return cobre estados (loading/data/error).
- [ ] Step 4: "O que é" — três decisões em qualquer hook customizado: (1) tupla vs objeto vs discriminated union; (2) generics vs anotação; (3) overloads quando há múltiplas formas de chamar.
- [ ] Step 5: "Como funciona" — code samples (mín 5):
  - `useToggle()` retornando `[boolean, () => void] as const`
  - Por que `as const` é necessário: sem ele, TS infere `(boolean | (() => void))[]` — perde posições
  - `useFetch<T>(url: string): { data: T | undefined; error: Error | null; isLoading: boolean }` — generic explicit
  - `useStorage` com overloads:
    ```typescript
    function useStorage<T>(key: string): [T | null, (value: T) => void];
    function useStorage<T>(key: string, defaultValue: T): [T, (value: T) => void];
    function useStorage<T>(key: string, defaultValue?: T) { ... }
    ```
  - Hook que retorna discriminated union: `useFetchUser(): { status: 'loading' } | { status: 'error'; error: Error } | { status: 'success'; data: User }`
- [ ] Step 6: "Na prática" — pattern observado em useSWR, useQuery (objeto com `data`/`error`/`isLoading`); pattern observado em `useToggle`/`useDisclosure` (tupla `as const`)
- [ ] Step 7: "Armadilhas" — esquecer `as const` em tupla; tentar inferir generic de retorno (não funciona — TS só infere de argumentos); over-engineering com discriminated union em hooks simples
- [ ] Step 8: "Em entrevista" — frase sobre tupla vs objeto e quando cada um faz sentido
- [ ] Step 9: "Veja também" — `[[01 - A tripla inferência]]`, `[[09 - Tipando reducers]]`, `[[12 - Generic components]]`, `[[TypeScript]]` (Generics)
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 07 - Tipando hooks customizados`)

---

### Task 9: Nota 08 — `08 - Tipando Context API.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/08 - Tipando Context API.md`

**Sources:**
- React docs — `createContext`, `useContext`
- Total TypeScript — Context patterns

- [ ] Step 1: Pré-research — confirmar API de Context em React 19
- [ ] Step 2: Frontmatter (tags: `context, providers, narrowing`)
- [ ] Step 3: TL;DR — `createContext<T>(null!)` é anti-pattern: vaza `null` para consumers. Padrão idiomático: `createContext<T | null>(null)` + custom hook que faz narrowing e throw se usado fora do provider. Provider deve memoizar `value` quando o objeto é construído.
- [ ] Step 4: "O que é" — Context API tem dois pontos de tipo: o tipo do valor e o tipo do default. Sem custom hook, o consumer precisa lidar com `null` toda vez.
- [ ] Step 5: "Como funciona" — code samples (mín 4):
  - Anti-pattern: `createContext<User>(null!)` — força non-null, pode crashar em runtime
  - Padrão idiomático com narrowing:
    ```typescript
    const UserContext = createContext<User | null>(null);
    function useUser(): User {
      const ctx = useContext(UserContext);
      if (!ctx) throw new Error('useUser must be inside UserProvider');
      return ctx;
    }
    ```
  - Provider com children tipado:
    ```typescript
    function UserProvider({ children }: { children: React.ReactNode }) {
      const [user, setUser] = useState<User | null>(null);
      const value = useMemo(() => ({ user, setUser }), [user]);
      return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
    }
    ```
  - Context com Action handlers (Provider expõe `{ user, login, logout }`)
- [ ] Step 6: "Na prática" — pattern observado em libs (React Router, Radix, Mantine): sempre custom hook para narrowing
- [ ] Step 7: "Armadilhas" — `null!` (force non-null); esquecer `useMemo` no value (re-render cascade); criar Context per-component sem necessidade (Context não é state management)
- [ ] Step 8: "Em entrevista" — frase sobre default null pattern + narrowing
- [ ] Step 9: "Veja também" — `[[05 - Tipando state e refs]]`, `[[09 - Tipando reducers]]`, `[[React]]`
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 08 - Tipando Context API`)

---

### Task 10: Nota 09 — `09 - Tipando reducers e state machines.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/09 - Tipando reducers e state machines.md`

**Sources:**
- React docs — `useReducer`
- [[TypeScript]] (seção "Discriminated unions")

- [ ] Step 1: Pré-research — confirmar pattern de exhaustiveness check (`never` no default)
- [ ] Step 2: Frontmatter (tags: `useReducer, discriminated-unions, state-machines`)
- [ ] Step 3: TL;DR — Actions tipadas como discriminated union pelo `type`. Reducer com `switch` força exhaustiveness via `never` no default. State machine completa (idle/loading/success/error) substitui múltiplos `useState` e força narrowing no render.
- [ ] Step 4: "O que é" — discriminated union: cada variant tem uma propriedade discriminante (`type`, `kind`, `status`) com literal type distinto. TS narroweia automaticamente.
- [ ] Step 5: "Como funciona" — code samples (mín 4):
  - Counter reducer com discriminated union:
    ```typescript
    type Action = { type: 'inc' } | { type: 'dec' } | { type: 'set'; value: number };
    function reducer(state: number, action: Action): number {
      switch (action.type) {
        case 'inc': return state + 1;
        case 'dec': return state - 1;
        case 'set': return action.value;  // narrowed para { type: 'set'; value: number }
        default: { const _: never = action; return state; }
      }
    }
    ```
  - State machine completa:
    ```typescript
    type State =
      | { status: 'idle' }
      | { status: 'loading' }
      | { status: 'success'; data: User }
      | { status: 'error'; error: Error };
    ```
  - Render com switch exhaustivo (TS força cobertura de todos os cases)
  - Comparação com 3 useState separados (estado inválido possível: `loading=true` E `data` populado)
- [ ] Step 6: "Na prática" — pattern observado em XState (state machines), TanStack Query (status discriminated)
- [ ] Step 7: "Armadilhas" — esquecer `default: never` (perde exhaustiveness); usar `string` em vez de literal no discriminator (não narroweia); estados inválidos modeláveis com booleans
- [ ] Step 8: "Em entrevista" — frase sobre exhaustiveness e estados inválidos
- [ ] Step 9: "Veja também" — `[[04 - interface vs type vs satisfies]]`, `[[07 - Tipando hooks customizados]]`, `[[11 - Tipando data fetching]]`, `[[TypeScript]]`
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 09 - Tipando reducers e state machines`)

---

### Task 11: Nota 10 — `10 - Tipando formulários.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/10 - Tipando formulários.md`

**Sources:**
- react-hook-form TypeScript guide: `https://react-hook-form.com/get-started#TypeScript`
- Zod docs: `https://zod.dev/`
- TanStack Form (alternativa moderna)
- [[TypeScript]] (seção "Runtime validation — Zod")

- [ ] Step 1: Pré-research — confirmar API atual de RHF + Zod (`zodResolver`)
- [ ] Step 2: Frontmatter (tags: `forms, validation, react-hook-form, zod`)
- [ ] Step 3: TL;DR — schema-driven typing: declare Zod schema, derive TS type via `z.infer<typeof schema>`. Integre com RHF via `zodResolver`. Erros viram discriminated union via `formState.errors`. Controlled inputs usam `value`/`onChange`; uncontrolled usam `register`.
- [ ] Step 4: "O que é" — formulários têm 3 fontes de tipo: schema (validação runtime), input handlers (events), state (RHF abstrai). Schema-driven elimina duplicação.
- [ ] Step 5: "Como funciona" — code samples (mín 5):
  - Schema Zod + `z.infer` para tipo
  - Setup RHF com `zodResolver`:
    ```typescript
    const schema = z.object({ email: z.string().email(), age: z.number().int().positive() });
    type FormData = z.infer<typeof schema>;
    const { register, handleSubmit, formState: { errors } } = useForm<FormData>({ resolver: zodResolver(schema) });
    ```
  - Submit handler tipado: `(data: FormData) => void`
  - Errors como discriminated union (cada campo tem `errors.<campo>?.message`)
  - Controlled vs uncontrolled — quando usar cada
- [ ] Step 6: "Na prática" — schema vivendo em arquivo separado, compartilhado com backend (single source of truth)
- [ ] Step 7: "Armadilhas" — declarar tipos manuais em vez de `z.infer`; `register` com nome errado (não detectado em TS, vira string); usar Yup em vez de Zod (Yup não infere tão bem)
- [ ] Step 8: "Em entrevista" — frase sobre schema-driven typing e single source of truth
- [ ] Step 9: "Veja também" — `[[06 - Tipando event handlers]]`, `[[09 - Tipando reducers]]`, `[[11 - Tipando data fetching]]`, `[[TypeScript]]`
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 10 - Tipando formulários`)

---

### Task 12: Nota 11 — `11 - Tipando data fetching.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/11 - Tipando data fetching.md`

**Sources:**
- TanStack Query TypeScript guide
- React docs — `use()` hook (React 19)
- Next.js 16 docs — Server Actions

- [ ] Step 1: Pré-research — confirmar API de Server Actions em Next 16 (typed?); confirmar `use()` em React 19
- [ ] Step 2: Frontmatter (tags: `data-fetching, react-query, tanstack, suspense, server-actions`)
- [ ] Step 3: TL;DR — `useQuery<TData, TError>` infere de `queryFn`. Validação runtime com Zod no boundary. Suspense + `use()` permite unwrap de Promise. Server Actions em Next 16 têm tipos derivados automaticamente.
- [ ] Step 4: "O que é" — três modelos de fetching tipado em 2026: client-side (RQ), suspense-driven (`use()`), server actions (Next).
- [ ] Step 5: "Como funciona" — code samples (mín 5):
  - `useQuery` com inferência:
    ```typescript
    const { data } = useQuery({
      queryKey: ['user', id],
      queryFn: () => fetchUser(id),  // (id: string) => Promise<User>
    });
    // data: User | undefined
    ```
  - Validação runtime com Zod no boundary do fetch
  - `use()` com Suspense (React 19):
    ```typescript
    function UserProfile({ promise }: { promise: Promise<User> }) {
      const user = use(promise);  // user: User
      return <div>{user.name}</div>;
    }
    ```
  - Server Action tipada (Next 16):
    ```typescript
    'use server';
    async function updateUser(data: UserUpdate): Promise<User> { ... }
    ```
  - Result type pattern para erros explícitos
- [ ] Step 6: "Na prática" — pattern observado: schema Zod no boundary, `useQuery` interno, Result type quando erros têm semântica
- [ ] Step 7: "Armadilhas" — `data` é `T | undefined` (loading); `error` é `Error | null` (não throw); confundir `useQuery` com `useSuspenseQuery` (este último não tem undefined)
- [ ] Step 8: "Em entrevista" — frase sobre runtime validation no boundary e por que TS sozinho não basta
- [ ] Step 9: "Veja também" — `[[09 - Tipando reducers]]` (state machines), `[[10 - Tipando formulários]]`, `[[TypeScript]]` (Zod)
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 11 - Tipando data fetching`)

---

## Wave 4 — Type-level avançado (Tasks 13-15)

3 notas. Depende de 04 (interface vs type) e 07 (hooks customizados).

### Task 13: Nota 12 — `12 - Generic components.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/12 - Generic components.md`

**Sources:**
- Total TypeScript — Generic Components
- React TypeScript Cheatsheet — Generic Components

- [ ] Step 1: Pré-research — confirmar workaround de vírgula em `.tsx` (`<T,>` ou `<T extends unknown>`)
- [ ] Step 2: Frontmatter (tags: `generics, components, advanced`)
- [ ] Step 3: TL;DR — function declarations suportam generics naturalmente (`function List<T>(props: { items: T[] })`). Arrow functions em `.tsx` precisam de hack (`<T,>` ou `<T extends unknown>`) por conflito com JSX. Constraint preserva inferência (`T extends { id: string }`).
- [ ] Step 4: "O que é" — generic component: tipo do componente depende de tipo do prop. Permite `<List<User> items={users} />` com inferência.
- [ ] Step 5: "Como funciona" — code samples (mín 5):
  - List genérico com function declaration:
    ```typescript
    function List<T>({ items, render }: { items: T[]; render: (item: T) => ReactNode }) {
      return <ul>{items.map((item, i) => <li key={i}>{render(item)}</li>)}</ul>;
    }
    ```
  - Mesmo componente com arrow function (workaround):
    ```typescript
    const List = <T,>({ items, render }: { items: T[]; render: (item: T) => ReactNode }) => (
      <ul>{items.map((item, i) => <li key={i}>{render(item)}</li>)}</ul>
    );
    // OU: <T extends unknown>
    ```
  - Uso com inferência: `<List items={users} render={u => u.name} />` — TS infere `T = User`
  - Constraint: `function List<T extends { id: string }>(...)` — força id presente
  - Select genérico com defaults: `function Select<T>(props: { options: T[]; getKey: (t: T) => string })`
- [ ] Step 6: "Na prática" — pattern observado em libs de tabela (`<Table<User> data={...} />`), select (`<Select<Option> options={...} />`)
- [ ] Step 7: "Armadilhas" — esquecer vírgula em arrow function (`<T>` é parsed como JSX); generics não inferem de retorno (só de argumentos); generics + `defaultValue` ficam estranhos
- [ ] Step 8: "Em entrevista" — frase sobre generic components e JSX ambiguity
- [ ] Step 9: "Veja também" — `[[04 - interface vs type]]`, `[[07 - Tipando hooks]]`, `[[13 - Polymorphic components]]`, `[[TypeScript]]`
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 12 - Generic components`)

---

### Task 14: Nota 13 — `13 - Polymorphic components com as prop.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/13 - Polymorphic components com as prop.md`

**Sources:**
- Total TypeScript — Polymorphic Components (módulo Advanced React with TypeScript)
- React 19 docs sobre `forwardRef` (não-mais-necessário em React 19)
- Material canônico a buscar (Step 1)

- [ ] Step 1: Pré-research — buscar artigo canônico sobre polymorphic components em 2026 (Total TypeScript tem; comunidade tem várias). **Não fabricar autores; verificar fontes.**
- [ ] Step 2: Frontmatter (tags: `polymorphic, as-prop, forwardRef, advanced`)
- [ ] Step 3: TL;DR — `<Box as="a" href="..." />` precisa que TS mude as props aceitas conforme `as`. Solução: generic component que recebe `as: ElementType`, combina com `ComponentPropsWithoutRef<typeof as>`. Em React 19, `ref` é prop normal — `forwardRef` não é mais necessário, mas ainda é o pattern em libs legacy.
- [ ] Step 4: "O que é" — polymorphic component: aceita prop `as` (string ou componente) que muda a tag/componente renderizado e o tipo das demais props.
- [ ] Step 5: "Como funciona" — code samples (mín 5):
  - Versão básica sem `as`: `Box` que sempre renderiza `div`
  - Versão com `as` simples (sem types corretos): bug — `href` é aceito mesmo quando `as="div"`
  - `ComponentPropsWithoutRef<T>`: extrai props de qualquer elemento/componente
  - Versão final com generics + ref forwarding (React 18 e anterior):
    ```typescript
    type PolymorphicProps<E extends ElementType> = {
      as?: E;
    } & Omit<ComponentPropsWithoutRef<E>, 'as'>;

    const Box = <E extends ElementType = 'div'>({ as, ...props }: PolymorphicProps<E>) => {
      const Component = as || 'div';
      return <Component {...props} />;
    };
    ```
  - Versão React 19 (ref como prop, sem forwardRef)
- [ ] Step 6: "Na prática" — pattern observado em Mantine (`Box` polymorphic), MUI (`Box` com `component` prop), Radix (mistura abordagens). Trade-off: complexidade de tipo vs UX.
- [ ] Step 7: "Armadilhas" — `as` não conflitante com props HTML (precisa `Omit<..., 'as'>`); ref tipo errado em forwardRef genérico; performance: cada render cria novo componente se `as` é função inline
- [ ] Step 8: "Em entrevista" — frase sobre `ComponentPropsWithoutRef` e generic + forwardRef. Pergunta clássica.
- [ ] Step 9: "Veja também" — `[[04 - interface vs type]]`, `[[05 - Tipando state e refs]]`, `[[12 - Generic components]]`, `[[14 - Compound components]]`
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 13 - Polymorphic components`)

---

### Task 15: Nota 14 — `14 - Compound components, slots, render props.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/14 - Compound components, slots, render props.md`

**Sources:**
- Radix UI docs (compound components canônicos)
- React docs — `Children.map`, `Children.toArray`, `cloneElement`

- [ ] Step 1: Pré-research — confirmar API de `Children` em React 19; conferir API de `<Slot>` do Radix
- [ ] Step 2: Frontmatter (tags: `compound-components, slots, render-props, composition, advanced`)
- [ ] Step 3: TL;DR — compound components compartilham state via Context interno (`<Tabs>`, `<Tabs.List>`, `<Tabs.Tab>`). Slots aceitam children específicos com validação de tipo. Render props (function-as-children) preservam inferência via generics.
- [ ] Step 4: "O que é" — três patterns de composição que tipam diferente: compound (state coordenado), slot (filho substitui wrapper), render prop (function como children).
- [ ] Step 5: "Como funciona" — code samples (mín 5):
  - Compound component com sub-componentes anexados: `Tabs.List = TabsList`
  - Context interno tipado para state coordenado:
    ```typescript
    type TabsContext = { value: string; setValue: (v: string) => void };
    const TabsCtx = createContext<TabsContext | null>(null);
    ```
  - Slot pattern (Radix-style): `<Slot>` que aceita um children e injeta props
  - Render prop tipado:
    ```typescript
    type DataLoaderProps<T> = {
      url: string;
      children: (state: { data: T | undefined; error: Error | null }) => ReactNode;
    };
    function DataLoader<T>({ url, children }: DataLoaderProps<T>) { ... }
    // Uso: <DataLoader<User> url="/me">{({ data }) => <h1>{data?.name}</h1>}</DataLoader>
    ```
  - Validação de children com `Children.map` (ex: rejeitar children que não sejam `<Tab>`)
- [ ] Step 6: "Na prática" — Radix usa Slot extensivamente; TanStack Table usa render props para flexibilidade
- [ ] Step 7: "Armadilhas" — anexar sub-component como propriedade requer tipo correto (`Tabs.List = TabsList as typeof TabsList`); `Children.map` perde tipos do children original; render props criam closures novas a cada render (usar `useCallback` se passar pra child memoizado)
- [ ] Step 8: "Em entrevista" — frase sobre os três patterns e quando cada um faz sentido
- [ ] Step 9: "Veja também" — `[[08 - Tipando Context API]]`, `[[12 - Generic components]]`, `[[13 - Polymorphic components]]`
- [ ] Step 10: Quality check + Commit (`feat(typescript-react): nota 14 - Compound components, slots, render props`)

---

## Wave 5 — Fechamento (Task 16)

### Task 16: Nota 15 — `15 - Armadilhas, tsconfig, ferramentas.md`

**Files:**
- Create: `JavaScript/Frontend/TypeScript com React/15 - Armadilhas, tsconfig, ferramentas.md`

**Sources:**
- React Compiler docs: `https://react.dev/learn/react-compiler`
- ts-reset (Matt Pocock): `https://github.com/total-typescript/ts-reset`
- ESLint plugin react-hooks: `https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks`
- ESLint plugin react-compiler: `https://www.npmjs.com/package/eslint-plugin-react-compiler`
- Vite TypeScript template: `https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react-ts`
- Next.js TypeScript docs: `https://nextjs.org/docs/app/api-reference/config/typescript`

- [ ] Step 1: Pré-research — refresh de todas as fontes (React 19 + Vite 8 + Next 16 + react-compiler + ts-reset)
- [ ] Step 2: Frontmatter (tags: `tsconfig, eslint, tooling, pitfalls, react-compiler, ts-reset`)
- [ ] Step 3: TL;DR — checklist de armadilhas TS+React (15 itens) + tsconfig específico React+Vite + tsconfig específico React+Next + ESLint plugins essenciais (`react-hooks`, `react-compiler`) + `ts-reset` para corrigir tipos built-in problemáticos.

Esta nota tem **estrutura tópica** (lista + blocos), não narrativa. Variação permitida do template.

- [ ] Step 4: Seção "O que é"

Nota agregadora — referência rápida para configurar projeto novo e checklist final antes de PR.

- [ ] Step 5: Seção "Armadilhas comuns" (lista numerada, 12-15 itens)

Cada armadilha com: nome, exemplo do erro, fix.

Lista mínima:
1. `useRef<T>(null!)` em vez de `useRef<T | null>(null)`
2. `as any` em casts de events em vez de tipo correto
3. Esquecer `as const` em retorno de hook (perde tupla)
4. `createContext<T>(null!)` em vez de `<T | null>(null)` + custom hook
5. `useState<T[]>([])` esquecido (fica `never[]`)
6. Generic em arrow function `.tsx` sem vírgula (`<T>` em vez de `<T,>`)
7. `e.target.value` em handlers que não são input (target é qualquer node)
8. `Object.keys(obj)` retorna `string[]` (não `(keyof T)[]`)
9. `JSON.parse` retorna `any` (sempre validar com Zod)
10. `catch (e)` é `unknown` (narrow com `instanceof Error`)
11. `React.FC` com generic (não funciona; use anotação direta)
12. Confundir `MouseEvent` (DOM) com `React.MouseEvent` (synthetic)
13. Children como `JSX.Element` em vez de `React.ReactNode`
14. Memoizar inline (`useMemo(() => ({}), [])` cria novo a cada deps mudarem)
15. `defaultProps` em function components (deprecated; use destructuring default)

- [ ] Step 6: Seção "tsconfig.json — React+Vite (2026)"

```json
{
  "compilerOptions": {
    "target": "ES2023",
    "lib": ["ES2023", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "allowImportingTsExtensions": true,
    "noEmit": true,
    "baseUrl": ".",
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["src"]
}
```

Comentários sobre flags específicas a React (jsx, paths, etc).

- [ ] Step 7: Seção "tsconfig.json — React+Next 16 (2026)"

Versão Next, com diferenças (`moduleResolution: "Bundler"`, `plugins`, etc). Pegar do template oficial.

- [ ] Step 8: Seção "ESLint essencial"

```jsonc
{
  "extends": [
    "plugin:react-hooks/recommended",
    "plugin:react-compiler/recommended"
  ],
  "plugins": ["@typescript-eslint", "react", "react-hooks", "react-compiler"]
}
```

- [ ] Step 9: Seção "ts-reset — corrige tipos built-in"

`ts-reset` corrige: `JSON.parse` retornando `any`, `Array.includes` aceitando qualquer string, `fetch` retornando `any`, etc. Adicionar em projeto novo.

- [ ] Step 10: Seção "Em entrevista" — frase pronta sobre tooling de TS+React em 2026 (strict mode + flags extras + react-hooks ESLint + ts-reset)

- [ ] Step 11: Seção "Veja também" — todas as outras 14 notas da trilha

- [ ] Step 12: Quality check (rubrica adaptada — esta nota é tópica, não narrativa) + Commit

```bash
git add "JavaScript/Frontend/TypeScript com React/15 - Armadilhas, tsconfig, ferramentas.md"
git commit -m "feat(typescript-react): nota 15 - Armadilhas, tsconfig, ferramentas"
```

---

## Wave 6 — Integração (Task 17)

### Task 17: Wikilinks cruzados + atualização de notas mãe

**Files:**
- Modify: `JavaScript/Frontend/TypeScript com React/TypeScript com React.md` (MOC — pass final de wikilinks)
- Modify: `JavaScript/Core/TypeScript.md` (seção "Veja também")
- Modify: `JavaScript/Frontend/React.md` (seção "Veja também")
- Modify: `Aprendizado/Trilha Frontend.md` (entrada para a trilha)

- [ ] **Step 1: Pass final no MOC**

Reler `TypeScript com React.md` e garantir que **todos** os wikilinks na seção "Comece por aqui" e nas rotas alternativas estão funcionando (notas existem com nomes corretos).

- [ ] **Step 2: Adicionar entrada em `JavaScript/Core/TypeScript.md`**

Localizar seção "Veja também" e adicionar:

```markdown
- [[TypeScript com React]] — trilha completa sobre a intersecção idiomática
```

- [ ] **Step 3: Adicionar entrada em `JavaScript/Frontend/React.md`**

Localizar seção "Veja também" e adicionar:

```markdown
- [[TypeScript com React]] — trilha completa sobre tipagem em React
```

- [ ] **Step 4: Adicionar entrada em `Aprendizado/Trilha Frontend.md`**

Adicionar seção (ou item em seção existente) com link para a trilha.

```markdown
## Trilhas relacionadas

- [[TypeScript com React]] — trilha de aprofundamento na intersecção TS+React
```

- [ ] **Step 5: Verificar que Quartz não quebra**

Confirmar que `JavaScript/Frontend/TypeScript com React/index.md` **não** existe (Quartz exige `index.md` apenas em raiz; em pastas comuns o MOC com mesmo nome basta). Se necessário verificar build do Quartz na pasta separada `josenaldo.github.io/`.

- [ ] **Step 6: Commit**

```bash
git add JavaScript/ Aprendizado/
git commit -m "feat(typescript-react): integrar trilha com notas mãe e Trilha Frontend"
```

---

## Critérios de aceitação (verificação final)

Antes de marcar a trilha como completa:

- [ ] 16 arquivos criados em `JavaScript/Frontend/TypeScript com React/`
- [ ] Todos com `publish: true` no frontmatter
- [ ] MOC tem "Comece por aqui" + 4 rotas alternativas + dataview
- [ ] Cada nota tem TL;DR + "O que é" + "Por que importa" + "Como funciona" + "Na prática" + "Armadilhas" + "Em entrevista" + "Veja também" (variações permitidas em 03 e 15)
- [ ] Cada nota tem ≥3 code samples em TS moderno (variação: 15 é tópica, não narrativa)
- [ ] Cada nota tem ≥2 wikilinks para outras notas + ≥1 wikilink para [[TypeScript]] ou [[React]]
- [ ] Notas mãe atualizadas com link para a trilha
- [ ] Trilha Frontend atualizada com link
- [ ] Zero atribuição de experiência pessoal ao autor
- [ ] Nenhuma alegação técnica não-trivial sem fonte ou code sample
- [ ] Quartz publica sem erro (verificar com build local antes de push)

---

## Notas operacionais

- **Pacing:** Wave 1 (Tasks 1-2) é bloqueante e crítica — fazer com cuidado, definir o tom da trilha. Tasks 3-15 podem ser paralelizadas em sessões diferentes (entre dias) sem perda de coerência, desde que o template padrão seja seguido.
- **Verificação de samples:** Code samples didáticos (sample 1-2 de cada nota) devem ser testados em [TypeScript Playground](https://www.typescriptlang.org/play). Para samples mais complexos (polymorphic components, generic components), incluir link para playground na própria nota.
- **Mudanças de versão:** Se alguma fonte sair do ar ou mudar API de forma incompatível durante a execução, **parar e perguntar antes de adaptar**. Nunca fabricar substituto plausível.
- **Skip de tasks:** Não há tasks puláveis. Cada nota tem propósito definido no spec; cortar nota = mudar escopo = atualizar spec primeiro.

