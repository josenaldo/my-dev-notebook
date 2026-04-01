---
title: "Estruturas de Dados"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - fundamentos
  - entrevista
publish: false
---

# Estruturas de Dados

Formas de organizar e armazenar dados que determinam a eficiência das operações sobre eles.

## O que é

Estruturas de dados são coleções organizadas com operações definidas e complexidades conhecidas. A escolha da estrutura certa impacta diretamente a performance e a legibilidade do sistema.

### Arrays e Listas

- **Array:** coleção de tamanho fixo com acesso por índice O(1). Em Java: `int[]`, `String[]`.
- **ArrayList / List:** array dinâmico que redimensiona automaticamente. Acesso O(1), inserção no final amortizado O(1), inserção no meio O(n).
- **LinkedList:** nós conectados por ponteiros. Inserção/remoção O(1) se tiver referência ao nó, mas acesso O(n). Útil para filas e deques.

### Hash Tables

- **HashMap / Object / Map:** associação chave-valor com lookup, inserção e remoção O(1) amortizado.
- **Colisões:** resolvidas por chaining (lista no bucket) ou open addressing (probing).
- **HashSet / Set:** conjunto sem duplicatas, baseado em hash table internamente.
- **Em Java:** `HashMap`, `HashSet`, `LinkedHashMap` (mantém ordem de inserção).
- **Em JS/TS:** `Map`, `Set`, `Object` (cuidado: Object só aceita string como chave).

### Trees

- **Binary Search Tree (BST):** cada nó tem no máximo 2 filhos. Filho esquerdo < pai < filho direito. Busca O(log n) no caso médio, O(n) no pior caso (desbalanceada).
- **Árvore balanceada (AVL, Red-Black):** garante O(log n) para todas as operações. Em Java: `TreeMap`, `TreeSet`.
- **Heap:** árvore binária completa onde o pai é sempre maior (max-heap) ou menor (min-heap) que os filhos. Base para priority queues. Em Java: `PriorityQueue`.
- **Trie:** árvore de prefixos para buscas em strings. Útil para autocomplete.

### Graphs

- **Representação:** lista de adjacência (mais comum, eficiente em memória) ou matriz de adjacência (acesso O(1) para verificar aresta).
- **Direcionados vs não-direcionados:** arestas com ou sem sentido.
- **Ponderados:** arestas com peso (distância, custo).
- **Algoritmos essenciais:** BFS (busca em largura), DFS (busca em profundidade), Dijkstra (caminho mais curto), topological sort.

### Stacks e Queues

- **Stack (Pilha):** LIFO (Last In, First Out). Operações push/pop O(1). Usado em: call stack, parsing de expressões, undo/redo, DFS.
- **Queue (Fila):** FIFO (First In, First Out). Operações enqueue/dequeue O(1). Usado em: BFS, message queues, scheduling.
- **Deque:** double-ended queue, permite inserção/remoção em ambas as pontas.
- **Priority Queue:** fila onde o elemento de maior (ou menor) prioridade sai primeiro. Implementada com heap.

## Complexidade (Big O)

| Estrutura | Busca | Inserção | Remoção | Espaço |
|-----------|-------|----------|---------|--------|
| Array | O(n) | O(1)* | O(n) | O(n) |
| ArrayList | O(n) | O(1) amortizado | O(n) | O(n) |
| LinkedList | O(n) | O(1)** | O(1)** | O(n) |
| HashMap | O(1) | O(1) | O(1) | O(n) |
| BST (balanceada) | O(log n) | O(log n) | O(log n) | O(n) |
| Heap | O(n) | O(log n) | O(log n) | O(n) |
| Stack | O(n) | O(1) | O(1) | O(n) |
| Queue | O(n) | O(1) | O(1) | O(n) |

\* No final do array. \*\* Com referência ao nó.

## Quando usar

- **Array/ArrayList:** acesso por índice, iteração sequencial, dados de tamanho previsível
- **LinkedList:** inserção/remoção frequente no início, implementação de filas
- **HashMap:** lookup por chave, cache, contagem de frequência, deduplicação
- **TreeMap/TreeSet:** quando precisa de ordenação natural + operações O(log n)
- **Heap/PriorityQueue:** top-K elementos, merge de listas ordenadas, scheduling
- **Stack:** parsing, backtracking, DFS
- **Queue:** BFS, processamento em ordem, buffers
- **Graph:** modelar relações entre entidades (redes sociais, rotas, dependências)

## Armadilhas comuns

- **Confundir ArrayList com LinkedList:** em Java, ArrayList é quase sempre melhor. LinkedList raramente é a escolha certa (cache miss, overhead de ponteiros).
- **Ignorar colisões em HashMaps:** objetos como chave precisam de `hashCode()` e `equals()` corretos em Java.
- **Usar a estrutura errada:** usar List quando precisa de Set (duplicatas), usar TreeMap quando HashMap basta (overhead de ordenação desnecessário).
- **Esquecer que HashMap não é thread-safe:** usar `ConcurrentHashMap` em cenários concorrentes.
- **Não considerar o fator de carga:** HashMap com muitas colisões degrada para O(n).

## Na prática (da minha experiência)

> Em projetos com Spring Boot, uso HashMap como cache em memória para dados que mudam raramente — por exemplo, configurações de sistema que são carregadas uma vez e consultadas frequentemente. Para filas de processamento assíncrono, a combinação de Kafka (como queue distribuída) com PriorityQueue local para ordenação por prioridade é um padrão que usei no MedEspecialista.

## How to explain in English

When discussing data structures in an interview, focus on trade-offs rather than definitions:

"The choice of data structure depends on the access pattern of the data. If I need constant-time lookups by a unique key, a hash map is the natural choice — in Java that's a `HashMap`, in TypeScript a `Map`. But if I also need the data to be sorted, I'd reach for a `TreeMap` which gives me O(log n) for all operations while maintaining natural ordering.

For processing items in a specific order, I'd use a queue — or a priority queue if priority matters. In distributed systems, this concept maps directly to message queues like Kafka, where we process events in order within a partition.

One thing I've learned is to be pragmatic about data structure choices. In most backend applications, 90% of the time you're using arrays, hash maps, and queues. The key is knowing when you need something more specialized — like a trie for autocomplete, or a graph for modeling relationships between entities."

### Key vocabulary

- lista encadeada → linked list: nós conectados por referências
- árvore binária de busca → binary search tree (BST): árvore ordenada para busca eficiente
- tabela hash → hash table / hash map: associação chave-valor com acesso O(1)
- fila de prioridade → priority queue: fila onde elementos saem por prioridade
- complexidade de tempo → time complexity: medida de eficiência em função do tamanho da entrada
- pilha → stack: estrutura LIFO
- fila → queue: estrutura FIFO
- grafo → graph: conjunto de vértices e arestas
- busca em largura → breadth-first search (BFS)
- busca em profundidade → depth-first search (DFS)

## Recursos

- [Visualgo](https://visualgo.net/) — visualização interativa de estruturas de dados
- [Big-O Cheat Sheet](https://bigocheatsheet.com/) — referência rápida de complexidades
- [Algorithms, Part I](https://www.coursera.org/learn/algorithms-part1) — curso Princeton (Coursera)
- [Algorithms, Part II](https://www.coursera.org/learn/algorithms-part2) — continuação

## Veja também

- [[Algoritmos]]
- [[Orientação a Objetos]]
- [[Java Fundamentals]] — Collections Framework
