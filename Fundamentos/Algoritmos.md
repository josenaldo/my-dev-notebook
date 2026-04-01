---
title: "Algoritmos"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - fundamentos
  - entrevista
publish: false
---

# Algoritmos

Sequências de passos bem definidas para resolver problemas computacionais, com foco em eficiência e correção.

## O que é

Um algoritmo é uma solução passo-a-passo para um problema. Em entrevistas, o que importa é demonstrar que você sabe **analisar a complexidade** (Big O), **escolher a abordagem certa** para cada problema, e **comunicar seu raciocínio** enquanto resolve.

## Como funciona

### Análise de complexidade (Big O)

Big O descreve o comportamento assintótico — como o tempo ou espaço crescem conforme a entrada cresce.

| Complexidade | Nome | Exemplo |
|-------------|------|---------|
| O(1) | Constante | Acesso a array por índice, HashMap.get() |
| O(log n) | Logarítmica | Binary search, operações em BST balanceada |
| O(n) | Linear | Percorrer array, busca sequencial |
| O(n log n) | Linearítmica | Merge sort, Tim sort (Arrays.sort em Java) |
| O(n²) | Quadrática | Bubble sort, dois loops aninhados |
| O(2ⁿ) | Exponencial | Subsets, combinações sem memoization |

**Regras práticas:**
- Ignore constantes: O(2n) = O(n)
- Domine o termo maior: O(n² + n) = O(n²)
- Considere o caso médio, não só o pior caso
- Espaço importa tanto quanto tempo

### Sorting

- **Arrays.sort() em Java:** Dual-Pivot Quicksort para primitivos, TimSort para objetos. O(n log n).
- **Array.sort() em JS:** TimSort na V8. Comparador custom com `(a, b) => a - b`.
- **Quando pensar em sorting:** se precisa de dados ordenados para binary search, merge, ou output ordenado.
- **Counting Sort / Radix Sort:** O(n) para inteiros em range limitado. Raro em entrevistas, mas bom saber que existe.

### Searching

- **Binary Search:** O(log n) em array ordenado. Template clássico:
  - Definir `left = 0, right = length - 1`
  - Enquanto `left <= right`: calcular `mid`, comparar, ajustar `left` ou `right`
  - Variações: buscar primeira/última ocorrência, buscar insertion point
- **Two Pointers:** dois ponteiros percorrendo array (geralmente ordenado). Útil para: soma de pares, remoção de duplicatas, palíndromos.
- **Sliding Window:** janela deslizante sobre array/string. Útil para: subarray máximo, substring sem repetição.

### Recursão e Dynamic Programming

- **Recursão:** solução que se chama a si mesma. Base case + recursive case.
- **Memoization (top-down DP):** cache de resultados para evitar recálculo. Transforma O(2ⁿ) em O(n).
- **Tabulation (bottom-up DP):** preencher tabela iterativamente. Geralmente mais eficiente em memória.
- **Quando usar DP:** problema tem subestrutura ótima (solução ótima do todo contém soluções ótimas das partes) e subproblemas sobrepostos.

### Algoritmos em Grafos

- **BFS:** explora nível por nível. Usa fila. Encontra caminho mais curto em grafos não-ponderados.
- **DFS:** explora em profundidade. Usa stack (ou recursão). Útil para: detecção de ciclo, topological sort, componentes conectados.
- **Dijkstra:** caminho mais curto em grafos ponderados (pesos positivos). Usa priority queue.
- **Topological Sort:** ordenação linear de vértices em DAG. Usado em: build systems, resolução de dependências.

## Quando usar

- **Binary Search:** qualquer busca em dados ordenados
- **Two Pointers:** problemas com arrays/strings ordenados
- **Sliding Window:** subarray/substring com restrições
- **BFS:** caminho mais curto, levels de árvore
- **DFS:** exploração completa, backtracking
- **DP:** otimização com subproblemas sobrepostos
- **Greedy:** escolha local ótima leva à solução global (nem sempre funciona — provar é difícil)

## Armadilhas comuns

- **Otimizar cedo demais:** em entrevistas, comece com a solução brute force, depois otimize. Mostra raciocínio.
- **Esquecer edge cases:** array vazio, um elemento, todos iguais, negativos, overflow.
- **Confundir BFS com DFS:** BFS usa fila (FIFO), DFS usa pilha (LIFO ou recursão).
- **DP sem identificar subproblemas:** antes de aplicar DP, defina claramente qual é o estado e a transição.
- **Não comunicar enquanto resolve:** em entrevista, narrar o raciocínio é tão importante quanto o código.

## Na prática (da minha experiência)

> No dia a dia de backend, os algoritmos mais usados são busca e ordenação em coleções. Com Java Streams e Collections Framework, raramente implemento algoritmos do zero — mas entender a complexidade me ajuda a escolher a estrutura certa. Por exemplo, ao processar grandes volumes de dados médicos no MedEspecialista, optei por HashMap para indexação O(1) em vez de busca linear, reduzindo tempo de processamento significativamente.

## How to explain in English

"In interviews, I approach algorithm problems methodically. I start by clarifying the problem — inputs, outputs, constraints, edge cases. Then I think out loud about possible approaches, starting with the brute force solution and its complexity, before optimizing.

For example, if asked to find a pair of numbers that sum to a target, I'd first mention the O(n²) brute force with two nested loops, then optimize to O(n) using a hash set to check for complements in a single pass. This shows I understand the trade-off between time and space.

In my day-to-day work, I don't implement sorting algorithms from scratch — I use the standard library. But understanding algorithmic complexity helps me make better architectural decisions. When I'm designing a search feature, knowing that binary search is O(log n) versus linear scan O(n) helps me decide whether to maintain a sorted index or add a database index."

### Key vocabulary

- busca binária → binary search: busca em dados ordenados dividindo pela metade
- programação dinâmica → dynamic programming (DP): otimização por cache de subproblemas
- complexidade de tempo → time complexity: Big O notation
- caso base → base case: condição de parada da recursão
- força bruta → brute force: solução ingênua que testa todas as possibilidades
- algoritmo guloso → greedy algorithm: escolha local ótima a cada passo
- ordenação → sorting: organizar elementos em ordem
- travessia → traversal: visitar todos os elementos de uma estrutura

## Recursos

### Cursos em vídeo

> [!info] Algorithms, Part I
> Offered by Princeton University.
> [https://www.coursera.org/learn/algorithms-part1](https://www.coursera.org/learn/algorithms-part1)

> [!info] Algorithms, Part II
> Offered by Princeton University.
> [https://www.coursera.org/learn/algorithms-part2](https://www.coursera.org/learn/algorithms-part2)

### Livros

- [Algorithms, 4th Edition](https://algs4.cs.princeton.edu/home/) — Robert Sedgewick and Kevin Wayne
- [Introduction to Programming in Python](https://introcs.cs.princeton.edu/python/home/) — Sedgewick, Wayne, and Dondero
- [An Introduction to the Analysis of Algorithms](https://aofa.cs.princeton.edu/home/) — Sedgewick and Flajolet
- [Analytic Combinatorics](https://ac.cs.princeton.edu/home/) — Flajolet and Sedgewick
- [Computer Science: An Interdisciplinary Approach](https://introcs.cs.princeton.edu/java/home/) — Sedgewick and Wayne

### Prática

- [LeetCode](https://leetcode.com/) — problemas de entrevista categorizados
- [NeetCode](https://neetcode.io/) — roadmap organizado por padrões

## Veja também

- [[Estruturas de Dados]]
- [[Coding Challenges Strategy]]
