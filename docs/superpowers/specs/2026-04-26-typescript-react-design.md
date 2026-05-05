---
title: "Spec — Trilha TypeScript com React"
date: 2026-04-26
author: Josenaldo
status: draft
type: spec
publish: false
---

# Spec — Trilha "TypeScript com React"

## 1. Contexto e motivação

Em 2026, TypeScript é a linguagem default em qualquer projeto React sério: assumido em starters do Vite e Next, exigido em job descriptions, mainstream nos ecossistemas TanStack/React Query/React Hook Form. A pergunta deixou de ser "devo usar TS com React?" e passou a ser "como uso TS com React **bem**?". E essa segunda pergunta tem respostas não-óbvias que distinguem um dev fluente de um dev que apenas "põe `: any` quando o compilador reclama".

O Codex Technomanticus já tem dois deep-dives massivos cobrindo as duas linguagens isoladamente:

- [[TypeScript]] (1500+ linhas) — sistema de tipos, generics, utility types, mapped types, conditional types, validação runtime com Zod, entrevista
- [[React]] (1645 linhas) — Hooks, state management, performance, RSC, formulários, testing, entrevista

Nas duas há trechos sobre a intersecção, mas distribuídos: a seção *"TypeScript em frontend (React)"* dentro de TypeScript.md cobre o básico de componentes/hooks tipados; React.md espalha exemplos TS pelos seus 1645 linhas. Falta um lugar canônico que aprofunde **especificamente** o que separa um senior de um pleno na intersecção: generics em componentes, polymorphic components, ref forwarding genérico, discriminated unions em props, tipagem idiomática de hooks customizados, Context API sem default null no consumer, formulários e data fetching com types preservados.

Essa intersecção também é onde mais aparecem perguntas de entrevista internacional ("Como você tipa um componente que aceita `as` prop?", "Como tipa um hook que retorna tupla?", "Por que `React.FC` saiu de moda?") — e onde a resposta superficial é facilmente identificada por um entrevistador experiente.

A trilha existe para preencher essa lacuna: cobrir **com profundidade** os patterns idiomáticos de TS+React modernos, partindo do mental model, passando pelos idiomas práticos do dia a dia, e terminando no ferramental type-level avançado que viabiliza componentes reutilizáveis robustos.

## 2. Objetivo

Produzir **15 notas atômicas + 1 MOC** (16 arquivos no total) em `JavaScript/Frontend/TypeScript com React/`, todas publicadas (`publish: true`), em PT-BR, cobrindo do mental model ao type-level avançado da intersecção TypeScript+React.

A trilha precisa ser:

- **Atômica** — cada nota linkável e citável separadamente
- **Híbrida em camadas** — TL;DR + corpo técnico, no padrão das demais notas do vault
- **Complementar, não duplicada** — pressupõe o leitor já leu (ou pode consultar) [[TypeScript]] e [[React]]; foca em dissolver a intersecção, não em re-explicar bases
- **Idiomática 2026** — React 19, TypeScript 5.x (com menção ao 7.0 chegando), Vite 8 / Next 16, ESM, React Compiler, RSC
- **Orientada a entrevistas internacionais** — cobre explicitamente os patterns mais perguntados em senior interviews

## 3. Escopo

### Em escopo

- 16 arquivos markdown na pasta `JavaScript/Frontend/TypeScript com React/`
- Todos com `publish: true` (Quartz publicará em josenaldo.github.io)
- Idioma: PT-BR exclusivamente nesta fase
- Pesquisa baseada em fontes primárias do ecossistema:
  - [TypeScript to know for React (Frontend Joy)](https://www.frontendjoy.com/p/typescript-to-know-for-react)
  - [Microsoft TypeScript-React-Conversion-Guide](https://github.com/Microsoft/TypeScript-React-Conversion-Guide)
  - [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
  - [Total TypeScript (Matt Pocock)](https://www.totaltypescript.com/) — em particular o módulo "TypeScript and React"
  - [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) — seção sobre JSX
  - [React docs](https://react.dev/reference/react) — referência canônica para hooks
  - Busca complementar (Andrew Branch, Sébastien Lorber, Dan Abramov posts) onde houver lacuna
- Wikilinks densos para [[TypeScript]], [[React]], [[JavaScript Fundamentals]], [[Testes em JavaScript]]

### Fora de escopo (trabalho futuro)

- **Re-explicar TS básico** — presume-se [[TypeScript]] como referência paralela. Quando uma nota precisar de generics, conditional types, mapped types, etc., linka para a seção da nota mãe em vez de re-explicar.
- **Re-explicar React básico** — mesmo princípio com [[React]].
- **Refactor das notas mãe** — a decisão de remover a seção "TypeScript em frontend (React)" de TypeScript.md (substituindo por link para esta trilha) é deliberadamente adiada. Se a trilha amadurecer, faz-se em projeto separado.
- **Versão em inglês** — derivação futura, em projeto separado, depois que a trilha PT-BR estiver estável.
- **Testes de componentes tipados** — vai para [[Testes em JavaScript]], que já é o lugar canônico de testing.
- **Migração JS→TS de codebases legados** — o Microsoft conversion guide aparece como fonte, mas a trilha não é um guia de migração. Quem quer migrar um app antigo lê o guia da Microsoft direto.
- **Build tooling profundo** (tsconfig completo, monorepo, projeto refs) — só o subset relevante a React aparece na nota 15.

## 4. Audiência e barra de qualidade

**Audiência primária:** dev fullstack que sabe JavaScript bem, conhece React em nível médio (já trabalhou com hooks, context, formulários), e sabe o suficiente de TypeScript para usar tipos básicos, mas que **não domina** os patterns idiomáticos da intersecção. O leitor pode ter lido (ou pode consultar) [[TypeScript]] e [[React]] — a trilha pressupõe esse acesso, mas não exige leitura completa antes.

**Audiência secundária:** dev em preparação para entrevistas internacionais que precisa demonstrar domínio fluente de TS+React. Para esse público, a trilha provê vocabulário, justificativas e exemplos defensivos para perguntas frequentes ("polymorphic components", "ref forwarding genérico", "context com narrowing", "discriminated unions em state").

**Barra de qualidade:** ao terminar de ler a trilha, o leitor deve conseguir:

1. Tipar qualquer componente novo sem recorrer a `as`/`any`
2. Escrever um hook customizado genérico com inferência preservada (ex: `useLocalStorage<T>` que infere `T` do default value)
3. Construir um polymorphic component com `forwardRef` (ex: `<Box as="a" href="..." />` com tipos corretos)
4. Explicar em entrevista, em inglês, por que `React.FC` saiu de moda e o que usar no lugar
5. Tipar Context API sem `null` vazando para o consumer (default null pattern + custom hook narrowing)
6. Reconhecer e tipar discriminated unions em state machines de UI (idle/loading/success/error)
7. Tipar formulários com RHF + Zod sem fricção, derivando o tipo do schema
8. Diferenciar `React.MouseEvent<HTMLButtonElement>` de `MouseEvent` (DOM nativo) e saber quando cada um aparece
9. Saber quando `as const` é necessário em retornos de hooks (tupla vs objeto)
10. Configurar tsconfig + ESLint para um projeto React+Vite ou React+Next moderno

## 5. Estrutura da trilha (16 arquivos)

### MOC

| #   | Arquivo                   | Propósito                                                                                                                                                                              |
| --- | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| —   | `TypeScript com React.md` | MOC central. Trilha sequencial recomendada + 4 rotas alternativas (entrevista / produção / library author / completa). Inclui orientação inicial sobre quando ler vs quando consultar. |

### Bloco 1 — Mental model (4 notas)

| #   | Arquivo                                                    | Propósito                       | Conteúdo-chave                                                                                                                                                                                                                                                                                                    |
| --- | ---------------------------------------------------------- | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 01  | `01 - A tripla inferência - props, state, hooks.md`        | Como o TS pensa em React        | As três fontes de inferência num componente: props (declaradas pelo dev), state (inferido do initializer ou anotado), hooks (return types das libs). O JSX é apenas açúcar para `React.createElement` — sabendo isso, vários "mistérios" de erro desaparecem. JSX intrinsic elements via `JSX.IntrinsicElements`. |
| 02  | `02 - Inferir vs anotar - quando deixar o TS trabalhar.md` | A regra prática                 | A regra: anote inputs (props, parâmetros, return types públicos), deixe inferir o resto. Quando inferência falha (initializers `null`, arrays vazios, objetos `{}`). `satisfies` como meio-termo. Por que `useState(null)` precisa de tipo explícito mas `useState(0)` não.                                       |
| 03  | `03 - Por que React.FC saiu de moda.md`                    | A história e a alternativa      | O que `React.FC` injetava (children implícito, `defaultProps`, problemas com generics). Por que a comunidade se afastou (PRs no DefinitelyTyped, posts do Lorber). A alternativa: anotar props como parâmetro. Quando ainda faz sentido (poucos casos). Como explicar em entrevista.                              |
| 04  | `04 - interface vs type vs satisfies para props.md`        | A decisão idiomática para props | `interface` (extensível, declaration merging) vs `type` (unions, mapped, conditional). A regra prática usada por libs do ecossistema (MUI, Mantine, Radix). `satisfies` para validar shape mantendo literal types. Quando preferir cada um. Convenção de nomenclatura (`Props` suffix).                           |

### Bloco 2 — Idiomas práticos (7 notas)

| #   | Arquivo                                     | Propósito                                      | Conteúdo-chave                                                                                                                                                                                                                                                                                                                      |
| --- | ------------------------------------------- | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 05  | `05 - Tipando state e refs.md`              | useState, useRef, ref objects vs callback refs | `useState<T>` quando inferência não basta. `useRef<HTMLInputElement>(null)` vs `useRef<number>(0)` — a diferença DOM ref vs mutable ref. `RefObject` vs `MutableRefObject`. Callback refs e quando aparecem. `useImperativeHandle` tipado. Forwarding ref básico (o genérico vai na nota 13).                                       |
| 06  | `06 - Tipando event handlers.md`            | Synthetic events, formulários nativos          | A hierarquia de eventos sintéticos do React (`SyntheticEvent`, `MouseEvent`, `ChangeEvent`, `FormEvent`). Diferença para eventos DOM nativos. Tipos genéricos (`React.ChangeEvent<HTMLInputElement>`). `currentTarget` vs `target` e por que importa. Padrões para handlers reutilizáveis.                                          |
| 07  | `07 - Tipando hooks customizados.md`        | Return types preservando inferência            | Por que `useToggle()` deve retornar tupla `[boolean, () => void] as const` em vez de objeto. Overloads para hooks com assinaturas múltiplas (ex: `useStorage(key)` vs `useStorage(key, default)`). Inferência de generics em hooks (ex: `useFetch<T>(url)` retornando `Result<T>`). Padrão de hook que retorna discriminated union. |
| 08  | `08 - Tipando Context API.md`               | Default null pattern + narrowing               | Por que `createContext<MyType>(null!)` é um anti-pattern (vaza null para consumers). O padrão idiomático: `createContext<MyType \| null>(null)` + custom hook que faz narrowing e throw se usado fora do provider. Provider props com children tipado. Memoização de value.                                                         |
| 09  | `09 - Tipando reducers e state machines.md` | Discriminated unions em ação                   | Action types como discriminated union (`{ type: 'increment' }` vs `{ type: 'set'; value: number }`). Reducer com switch exhaustivo. State machine completa (idle/loading/success/error) e como o TS força o switch a cobrir todos os casos. Por que isso é melhor do que múltiplos `useState`.                                      |
| 10  | `10 - Tipando formulários.md`               | RHF + Zod, controlled vs uncontrolled          | Schema-driven typing: declarar schema Zod, derivar tipo com `z.infer`. Integração com React Hook Form via `zodResolver`. Tipando inputs nativos vs custom. Controlled (`value` + `onChange`) vs uncontrolled (`ref`). Form events vs change events. Errors como discriminated union.                                                |
| 11  | `11 - Tipando data fetching.md`             | React Query, Suspense, Server Actions          | `useQuery<TData, TError>` com inferência. Validação runtime com Zod no boundary. Suspense + tipos (Promise unwrapping com `use()`). Server Actions tipadas (Next.js 16 / React 19). Padrões para erros (Result type vs throw).                                                                                                      |

### Bloco 3 — Type-level avançado (3 notas)

| #   | Arquivo                                            | Propósito                                       | Conteúdo-chave                                                                                                                                                                                                                                                                                    |
| --- | -------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 12  | `12 - Generic components.md`                       | Componentes que preservam inferência            | `function List<T>(props: { items: T[]; render: (item: T) => ReactNode })`. Sintaxe arrow vs function declaration (limitações). Por que `<Select<User> options={...} />` precisa de virgula extra em `.tsx`. Casos canônicos: `List`, `Select`, `Table`. Constraints (`T extends { id: string }`). |
| 13  | `13 - Polymorphic components com as prop.md`       | `<Box as="a" href="..." />` tipado corretamente | O problema: como deixar `as` mudar o tipo de props aceitas. `ComponentProps<T>`, `ComponentPropsWithoutRef<T>`. Pattern com generic + `forwardRef`. Versão final: helper `PolymorphicComponentProps<E, P>`. Trade-off entre complexidade e UX. Alternativa moderna (React 19 sem `forwardRef`).   |
| 14  | `14 - Compound components, slots, render props.md` | Composição tipada                               | Compound components (`<Tabs>`, `<Tabs.List>`, `<Tabs.Tab>`) com tipos coordenados via Context. Slots pattern (`Children.map`, `Children.toArray`, validação de tipo de filho). Render props e function-as-children com inferência preservada. Comparação com generic components.                  |

### Bloco 4 — Fechamento (1 nota)

| #   | Arquivo                                     | Propósito                       | Conteúdo-chave                                                                                                                                                                                                                                                                                                                          |
| --- | ------------------------------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 15  | `15 - Armadilhas, tsconfig, ferramentas.md` | O que checar antes de fechar PR | Armadilhas comuns (`!` em `useRef`, `as` em casts de events, esquecer `as const` em retornos de hook, default null vazado de Context, generic perdido em arrow components). tsconfig específico React+Vite e React+Next 16. ESLint rules (`react-hooks`, `react-compiler`). `ts-reset` para corrigir tipos built-in. Cheatsheet visual. |

## 6. Padrão estrutural por nota

Toda nota segue o padrão das notas do vault:

```markdown
---
title: "<título>"
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
aliases:
  - <alias 1>
  - <alias 2>
---

# <título>

> [!abstract] TL;DR
> <2-4 linhas resumindo a nota>

## O que é
<conceito central, 1-3 parágrafos>

## Por que importa
<motivação, casos de uso, contraste com alternativas>

## Como funciona
<mecânica técnica, código, edge cases>

## Na prática
<exemplo realista, idealmente baseado em situação de produção>

## Armadilhas
<gotchas específicos da nota>

## Em entrevista
<como explicar o conceito em inglês, frases-chave>

## Veja também
<wikilinks para outras notas da trilha + notas mãe>
```

Variações são permitidas (ex: nota 03 sobre `React.FC` pode ter "História" no lugar de "Como funciona"). O esqueleto serve de referência, não de gabarito rígido.

**Tamanho típico:** 200-500 linhas por nota, alinhado com o padrão de memoria-agentes.

## 7. Rotas alternativas (no MOC)

### Rota entrevista

Para preparar perguntas frequentes em entrevista internacional sobre TS+React:
03 → 04 → 07 → 09 → 12 → 13

### Rota produção

Para escrever bem no dia a dia, sem se preocupar com type-level avançado:
01 → 02 → 05 → 06 → 08 → 10 → 11 → 15

### Rota library author

Para quem escreve componentes reutilizáveis em design system, biblioteca pública, ou pacote interno:
04 → 07 → 12 → 13 → 14

### Rota completa

Sequencial 01 → 15. Recomendada na primeira leitura.

## 8. Fontes principais

### Apontadas pelo autor

- [TypeScript to know for React — Frontend Joy](https://www.frontendjoy.com/p/typescript-to-know-for-react)
- [Microsoft TypeScript-React-Conversion-Guide](https://github.com/Microsoft/TypeScript-React-Conversion-Guide)

### Complementares (já no radar do autor)

- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/) — referência da comunidade, atualizada
- [Total TypeScript — Matt Pocock](https://www.totaltypescript.com/) — em especial o módulo "Advanced React with TypeScript"
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/jsx.html) — seção JSX
- [React docs (react.dev)](https://react.dev/reference/react) — referência canônica para tipos de hooks
- [react-hook-form docs](https://react-hook-form.com/get-started#TypeScript) — para a nota 10
- [TanStack Query TS guide](https://tanstack.com/query/latest/docs/framework/react/typescript) — para a nota 11

### A buscar conforme necessidade

- Discussão histórica sobre remoção de `React.FC` em projetos populares (CRA, Next.js) — fontes a verificar durante a redação da nota 03
- Material canônico sobre polymorphic components em 2026 — buscar durante redação da nota 13 (Total TypeScript tem módulo dedicado; a comunidade tem várias implementações de referência)
- [React Compiler docs](https://react.dev/learn/react-compiler) (nota 15)
- [ts-reset](https://github.com/total-typescript/ts-reset) (nota 15)
- Posts complementares sobre patterns específicos (busca dirigida durante redação de cada nota)

## 9. Decisões editoriais

- **Idioma:** PT-BR exclusivamente nesta fase
- **Tom:** técnico mas acessível, com TL;DR sempre presente; padrão das notas do vault
- **Versões assumidas:** React 19, TypeScript 5.x (com menção pontual ao 7.0 quando relevante para performance), Vite 8, Next 16
- **Frontmatter:** `publish: true`, `status: seedling`, tags `[typescript, react, typescript-react, frontend]` em todas
- **Wikilinks:** densidade alta, especialmente para [[TypeScript]] e [[React]] como referências paralelas
- **Code samples:** sempre TypeScript (não JS), sempre com tipos explícitos quando didáticos, comentários em PT-BR
- **Inglês:** apenas em "Em entrevista" (frases prontas) e em vocabulário técnico que não tem tradução natural

## 10. Riscos e mitigações

| Risco                                                                   | Mitigação                                                                                                                        |
| ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Sobreposição com `TypeScript.md` ou `React.md`                          | Cada nota tem 1-2 wikilinks explícitos para a seção mãe; corpo da nota assume conhecimento da mãe e foca na intersecção          |
| Perda de relevância com mudança de React (ex: `forwardRef` em React 19) | Nota 13 cobre tanto o pattern legacy (`forwardRef`) quanto o moderno (sem `forwardRef`); nota 15 marca explicitamente o que muda |
| Notas ficarem genéricas demais (estilo "tutorial qualquer")             | Toda nota tem seção "Na prática" com exemplo concreto (idealmente do contexto pessoal do autor), e seção "Em entrevista"         |
| Exemplos ficarem desatualizados                                         | Versões assumidas declaradas no spec; frontmatter `updated` atualizado a cada revisão; nota 15 funciona como "rolling reference" |
| Ambição: 16 notas é muito                                               | Trilha pode ser executada em fases (Bloco 1 primeiro, depois 2, depois 3); MOC pode ser publicado com notas pendentes marcadas   |

## 11. Critérios de aceitação

A trilha está completa quando:

1. Todas as 16 notas existem em `JavaScript/Frontend/TypeScript com React/`
2. Todas têm frontmatter completo com `publish: true`
3. MOC tem seção "Comece por aqui" com 15 notas linkadas em ordem + 4 rotas alternativas + dataview de "Todas as notas"
4. Cada nota tem TL;DR no callout `[!abstract]` e seções padrão (O que é, Por que importa, Como funciona, Na prática, Em entrevista, Veja também)
5. Cada nota tem ao menos 2 wikilinks para outras notas (incluindo as notas mãe quando aplicável)
6. Quartz publica corretamente (sem erros de build) em josenaldo.github.io
7. Pelo menos 3 notas têm exemplo de código testado em TypeScript Playground com link
8. Trilha é citável em LinkedIn / blog post como referência única

## 12. Plano de execução

O plano detalhado de execução vai em `docs/superpowers/plans/2026-04-26-typescript-react-execution.md` (gerado pela skill `writing-plans` após aprovação deste spec).

A ordem de execução recomendada:

1. MOC (esqueleto sem links ainda) + 01 + 02 (mental model fundamental)
2. 03 + 04 (fechar mental model)
3. 05 → 11 sequencialmente (idiomas práticos, podem ser paralelizados se necessário)
4. 12 → 14 (type-level avançado, depende de 04 e 07 estarem prontos)
5. 15 (fechamento, agrega de todas as outras)
6. Pass final no MOC para inserir todos os wikilinks finais
