---
title: "React"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - javascript
  - frontend
  - entrevista
publish: false
---

# React

Biblioteca JavaScript para construir interfaces de usuário declarativas e componentizadas.

## O que é

React é uma biblioteca (não framework) para UI, baseada em componentes, virtual DOM, e fluxo de dados unidirecional. Para entrevistas, o foco é em: hooks, state management, rendering behavior, performance, e patterns de composição.

## Como funciona

### Componentes e JSX

```tsx
interface PatientCardProps {
  patient: Patient;
  onSelect: (id: string) => void;
}

function PatientCard({ patient, onSelect }: PatientCardProps) {
  return (
    <div className="card" onClick={() => onSelect(patient.id)}>
      <h3>{patient.name}</h3>
      <span>{patient.specialty}</span>
    </div>
  );
}
```

- **Function components:** padrão atual. Class components são legados.
- **JSX:** syntax extension que compila para `React.createElement()`.
- **Props:** dados passados do pai para o filho (imutáveis).

### Hooks essenciais

```tsx
// Estado local
const [count, setCount] = useState(0);

// Side effects
useEffect(() => {
  const sub = api.subscribe(handler);
  return () => sub.unsubscribe(); // cleanup
}, [dependency]); // re-executa quando dependency muda

// Referência mutável (não causa re-render)
const inputRef = useRef<HTMLInputElement>(null);

// Memoization
const expensive = useMemo(() => computeValue(data), [data]);
const stableCallback = useCallback((id: string) => fetch(id), []);

// Context
const theme = useContext(ThemeContext);
```

### Rendering e Reconciliation

1. State/props mudam → React re-renderiza o componente
2. Virtual DOM diff compara árvore antiga com nova
3. Apenas mudanças reais são aplicadas ao DOM

**Re-render ≠ DOM update.** Re-render é barato; DOM updates são caros. React otimiza minimizando DOM updates.

### State Management

| Abordagem | Quando usar |
| --- | --- |
| `useState` | Estado local de um componente |
| `useReducer` | Estado complexo com múltiplas transições |
| Context + `useReducer` | Estado compartilhado em árvore pequena |
| TanStack Query | Server state (cache, loading, error, refetch) |
| Zustand | Client state global simples |
| Redux Toolkit | Client state global complexo (legado em muitos projetos) |

**Distinção crítica:** **server state** (dados do backend — cache, sync) vs **client state** (UI state — modais, filtros). TanStack Query para server state, Zustand/Context para client state.

### Patterns

**Composition over configuration:**

```tsx
// Ruim: prop drilling e configuração via props
<DataTable columns={[...]} sortable filterable paginated />

// Bom: composição de componentes
<DataTable data={patients}>
  <DataTable.Header>
    <SortableColumn field="name" />
    <FilterableColumn field="specialty" />
  </DataTable.Header>
  <DataTable.Pagination pageSize={20} />
</DataTable>
```

**Custom hooks:** extrair lógica reutilizável.

```tsx
function usePatients(specialty: string) {
  return useQuery({
    queryKey: ["patients", specialty],
    queryFn: () => api.getPatients({ specialty }),
  });
}
```

### Performance

- **`React.memo()`:** evita re-render quando props não mudaram (shallow compare)
- **`useMemo` / `useCallback`:** memoizar valores e funções caras
- **Lazy loading:** `React.lazy()` + `Suspense` para code splitting
- **Virtualization:** `react-window` ou `@tanstack/react-virtual` para listas longas
- **Key prop:** usar IDs estáveis (não índice do array) para listas

## Quando usar

- **React:** SPAs, dashboards, apps interativos complexos
- **Next.js:** quando precisa de SSR, SSG, API routes, ou SEO
- **TanStack Query:** qualquer comunicação com API (substitui useEffect + useState para fetching)
- **Zustand:** estado global simples sem boilerplate do Redux

## Armadilhas comuns

- **useEffect para fetching:** usar TanStack Query ou SWR. useEffect não gerencia cache, loading, error, race conditions.
- **Estado derivado em useState:** se um valor pode ser calculado a partir de outro estado, use `useMemo`, não outro `useState`.
- **Memoização prematura:** `React.memo`, `useMemo`, `useCallback` adicionam complexidade. Otimizar quando medir que é necessário.
- **Prop drilling profundo:** contexto ou composição são melhores que passar props por 5+ níveis.
- **useEffect sem cleanup:** subscriptions, timers, event listeners precisam de cleanup no return.
- **Key com índice de array:** causa bugs em listas que reordenam ou filtram.

## Na prática (da minha experiência)

> No frontend do MedEspecialista, uso React com TypeScript, TanStack Query para comunicação com a API Spring Boot, e Zustand para estado de UI (sidebar, modais, filtros). O componente de agenda médica é o mais complexo — usa virtualização para renderizar centenas de slots de horário sem degradar performance. Custom hooks como `useAppointments(date, doctorId)` encapsulam toda a lógica de fetch, cache e invalidação.

## How to explain in English

"React is my primary frontend library. I build applications with function components and hooks, using TypeScript for type safety on props and state. My architecture separates server state from client state — I use TanStack Query for data fetching, caching, and synchronization with the backend, and Zustand for UI state like modal visibility and filter selections.

For component design, I favor composition over configuration. Instead of a monolithic component with dozens of props, I create composable pieces that can be assembled flexibly. This makes components easier to test, reuse, and modify independently.

Performance optimization is something I approach with measurement first. React's rendering is already efficient — unnecessary memoization adds complexity without benefit. I reach for `React.memo` and `useMemo` only when profiling shows a specific component is re-rendering expensively. For truly large datasets, I use virtualization to render only visible items.

One thing I emphasize is proper data fetching. Using `useEffect` with `useState` for API calls is an anti-pattern — it doesn't handle caching, race conditions, loading states, or error recovery. TanStack Query solves all of these problems declaratively."

### Key vocabulary

- componente → component: unidade de UI reutilizável
- estado → state: dados que o componente gerencia
- propriedades → props: dados passados pelo componente pai
- gancho → hook: função que adiciona funcionalidade a componentes
- renderização → rendering: processo de gerar a UI
- reconciliação → reconciliation: diffing do virtual DOM
- divisão de código → code splitting: carregar código sob demanda
- estado do servidor → server state: dados do backend cacheados no frontend

## Recursos

- [React Docs](https://react.dev/) — documentação oficial (nova)
- [TanStack Query](https://tanstack.com/query/) — data fetching
- [Zustand](https://zustand-demo.pmnd.rs/) — state management
- [[Trilha Frontend]] — trilha de aprendizado

## Veja também

- [[JavaScript Fundamentals]]
- [[TypeScript]]
- [[Node.js]]
- [[API Design]]
