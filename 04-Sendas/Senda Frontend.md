---
type: trail
title: "Senda Frontend"
domain: "[[03-Domínios/Frontend/index]]"
maturity: minimal
status: active
publish: true
created: 2024-01-01
updated: 2026-05-04
tags:
  - senda
  - frontend
---

# Senda Frontend

> Caminho de estudo de engenharia frontend. Atravessa fundamentos de linguagem (JavaScript core, TypeScript), libs e frameworks (React e ecossistema), validação, networking, debugging, ferramentas, e a disciplina de engenharia frontend (arquitetura, padrões, performance, acessibilidade).

## Pré-requisitos

(deixar vazio inicialmente; popular conforme necessário)

## Sequência

1. [[03-Domínios/Frontend/index|Frontend (engenharia)]]
2. [[03-Domínios/JavaScript/JavaScript|JavaScript]]
3. [[03-Domínios/TypeScript/index|TypeScript]]
4. [[03-Domínios/React/index|React]]
5. [[03-Domínios/React/MUI|Material UI (MUI)]]
6. [[03-Domínios/React/Mantine|Mantine]]
7. [[03-Domínios/React/Ícones|Ícones em React]]
8. [[03-Domínios/React/TanStack Query|TanStack Query]]
9. [[03-Domínios/React/TanStack Form|TanStack Form]]
10. [[03-Domínios/React/React Hook Form|React Hook Form]]
11. [[03-Domínios/React/Next.js|Next.js]]
12. [[03-Domínios/Frontend/Validação/index|Validação no Frontend]]
13. [[03-Domínios/Frontend/Validação/Zod|Zod]]
14. [[03-Domínios/Frontend/Validação/Yup|Yup]]
15. [[03-Domínios/Frontend/Validação/Joi|Joi]]
16. [[03-Domínios/Frontend/Networking/index|Networking no Frontend]]
17. [[03-Domínios/Frontend/Networking/Axios|Axios]]
18. [[03-Domínios/Frontend/Networking/Fetch|Fetch API]]
19. [[03-Domínios/React/React Data Table|React Data Table Component]]
20. [[03-Domínios/React/React Admin|React Admin]]
21. [[03-Domínios/React/Charts/index|Charts em React]]
22. [[03-Domínios/React/Charts/Lightweight Charts|TradingView Lightweight Charts]]
23. [[03-Domínios/React/Charts/ApexCharts|ApexCharts]]
24. [[03-Domínios/React/Charts/Recharts|Recharts]]
25. [[03-Domínios/Frontend/Debugging|Frontend Debugging]]
26. [[03-Domínios/Ferramentas/Vite|Vite]]
27. [[03-Domínios/Ferramentas/Monorepos|Monorepos]]

## Trilhas relacionadas

- [[TypeScript com React]] — aprofundamento da intersecção TS+React (15 notas + MOC)

## Progresso

```dataview
TABLE WITHOUT ID
  link(file.path, regexreplace(file.folder, ".*/", "") + "/" + file.name) AS "Nota",
  default(progresso, "pendente") AS "Status"
FROM outgoing([[]])
WHERE file.path != this.file.path AND contains(file.path, "03-Domínios/")
SORT file.folder ASC, file.name ASC
```

**Resumo:**

```dataview
TABLE WITHOUT ID
  length(rows) AS "Total",
  length(filter(rows, (r) => default(r.progresso, "pendente") = "feito")) AS "Feitas",
  length(filter(rows, (r) => default(r.progresso, "pendente") = "andamento")) AS "Em andamento",
  length(filter(rows, (r) => default(r.progresso, "pendente") = "pausado")) AS "Pausadas",
  length(filter(rows, (r) => default(r.progresso, "pendente") = "pendente")) AS "Pendentes"
FROM outgoing([[]])
WHERE file.path != this.file.path AND contains(file.path, "03-Domínios/")
GROUP BY true
```
