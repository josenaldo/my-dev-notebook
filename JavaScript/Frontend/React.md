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

### Arquitetura de projeto

Estrutura de diretórios recomendada para projetos médios/grandes:

```text
src/
├── components/        ← componentes reutilizáveis (Button, Card, Modal)
│   └── ui/            ← componentes de design system
├── pages/             ← páginas/views (uma por rota)
├── hooks/             ← custom hooks reutilizáveis
├── services/          ← API layer (chamadas HTTP)
├── stores/            ← state management (Zustand stores)
├── types/             ← TypeScript types/interfaces compartilhados
├── utils/             ← funções utilitárias puras
├── contexts/          ← React contexts (theme, auth, i18n)
└── assets/            ← imagens, fontes, estilos globais
```

**Princípios:**

- **Separar server state de client state:** TanStack Query para dados do backend, Zustand/Context para estado de UI
- **Custom hooks para lógica de negócio:** cada feature tem seu hook (`usePatients`, `useAppointments`)
- **Múltiplos contexts pequenos:** não um "AppContext" monolítico. Um para auth, outro para theme, etc.
- **Componentes sem lógica de fetch:** componentes recebem dados via hooks, não fazem fetch internamente
- **Importações absolutas:** configurar `baseUrl` no tsconfig para evitar `../../../`

> **Fontes:**
> - [React Architecture Patterns](https://www.etatvasoft.com/blog/react-architecture-patterns/)
> - [Modularizing React Apps — Martin Fowler](https://martinfowler.com/articles/modularizing-react-apps.html)

### Formulários e Validação

**React Hook Form** — formulários performáticos (uncontrolled por padrão):

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const schema = z.object({
  name: z.string().min(1, "Nome obrigatório"),
  email: z.string().email("Email inválido"),
  age: z.number().min(18, "Deve ser maior de idade"),
});

type FormData = z.infer<typeof schema>;

function PatientForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data: FormData) => createPatient(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("name")} />
      {errors.name && <span>{errors.name.message}</span>}
      <button type="submit">Salvar</button>
    </form>
  );
}
```

**Bibliotecas de validação:**

| Lib | Estilo | Use case |
| --- | --- | --- |
| Zod | Schema-first, TypeScript-native | Preferido com React Hook Form + TS |
| Yup | Encadeamento fluente | Popular, mais antigo que Zod |
| Joi | API rica, origin Node.js | Backend-first, também funciona no front |

> **Fontes:**
> - [React Hook Form](https://react-hook-form.com/)
> - [TanStack Form](https://tanstack.com/form/latest)
> - [Zod](https://zod.dev/) — validação TypeScript-first
> - [Yup](https://github.com/jquense/yup)

### HTTP e API Layer

**Axios** — HTTP client com interceptors, cancel tokens, e configuração global:

```tsx
// services/api.ts — configuração centralizada
import axios from "axios";

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) redirectToLogin();
    return Promise.reject(error);
  }
);
```

**TanStack Query + Axios** — o padrão recomendado:

```tsx
// hooks/usePatients.ts
export function usePatients(specialty: string) {
  return useQuery({
    queryKey: ["patients", specialty],
    queryFn: () => api.get<Patient[]>(`/patients?specialty=${specialty}`).then(r => r.data),
    staleTime: 5 * 60 * 1000, // 5 min cache
  });
}

// hooks/useCreatePatient.ts
export function useCreatePatient() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: CreatePatientDTO) => api.post("/patients", data),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ["patients"] }),
  });
}
```

> **Fontes:**
> - [TanStack Query](https://tanstack.com/)
> - [React Query useMutation](https://profy.dev/article/react-query-usemutation)
> - [React REST API patterns](https://profy.dev/article/react-rest-api)

### Ecossistema e Tooling

| Ferramenta | Categoria | O que faz |
| --- | --- | --- |
| Vite | Build tool | Dev server rápido, HMR, build otimizado (substitui CRA) |
| Next.js | Meta-framework | SSR, SSG, API routes, file-based routing |
| React Router | Routing | Navegação SPA, nested routes, loaders |
| TanStack Query | Server state | Fetch, cache, sync, optimistic updates |
| Zustand | Client state | Estado global minimalista |
| React Hook Form | Forms | Formulários performáticos |
| Zod | Validação | Schema validation TypeScript-first |
| Axios | HTTP | Client HTTP com interceptors |
| React Admin | Admin panels | CRUD admin out-of-the-box |
| Tabler Icons | Ícones | Biblioteca de ícones open-source |

**Migração CRA → Vite:** essencial para projetos existentes. Vite é 10-100x mais rápido no dev server.

> **Fontes:**
> - [Next.js](https://nextjs.org/)
> - [React Admin](https://marmelab.com/react-admin/)
> - [Migrar de CRA para Vite](https://dev.to/ajeetraina/how-to-migrate-from-create-react-app-to-vite-3b8m)
> - [Tabler Icons](https://tabler.io/icons)
> - [Lightweight Charts (gráficos)](https://www.tradingview.com/lightweight-charts/)

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

- [React Docs](https://react.dev/) — documentação oficial
- [React Reference](https://react.dev/reference/react) — API reference
- [TanStack Query](https://tanstack.com/query/) — data fetching
- [Zustand](https://zustand-demo.pmnd.rs/) — state management
- [React Hook Form](https://react-hook-form.com/) — formulários
- [React Components Demystified](https://dev.to/vyan/react-components-demystified-your-ultimate-guide-from-newbie-to-ninja-3l1n)
- [Frontend Debugging 101](https://lukeberrypi.vercel.app/articles/frontend-debugging-101)
- [Frontend Design Patterns](https://www.netguru.com/blog/frontend-design-patterns)
- [[Trilha Frontend]] — trilha de aprendizado

## Veja também

- [[JavaScript Fundamentals]]
- [[TypeScript]]
- [[Node.js]]
- [[HTML e CSS]]
- [[Material UI]]
- [[Mantine]]
- [[API Design]]
