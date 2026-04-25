---
title: "Spec — Trilha Memória de Agentes (LLM Wiki Pattern)"
date: 2026-04-25
author: Josenaldo
status: draft
type: spec
publish: false
---

# Spec — Trilha "Memória de Agentes (LLM Wiki Pattern)"

## 1. Contexto e motivação

Em 3 de abril de 2026 Andrej Karpathy publicou no X (e em seguida via gist) o que ele chama de **"LLM Wiki" pattern** — um esquema em que o LLM não apenas consulta um corpo de conhecimento, mas **constrói e mantém ativamente** uma wiki interlinkada em markdown a partir de fontes brutas. A repercussão foi imediata: VentureBeat publicou que "bypassa RAG", surgiram dezenas de implementações e tutoriais em poucas semanas, e o tema se cruzou com um campo acadêmico já em ebulição (memória de agentes), gerando confusão entre "o que é hype recente do Twitter" e "o que é arquitetura técnica madura".

Este projeto entrega uma trilha de notas em português brasileiro, hospedada no vault Codex Technomanticus, capaz de:

1. Levar um leitor leigo (que sabe usar Obsidian e tem alguma noção de LLM) ao entendimento profundo do tema
2. Servir como base de discurso público (artigos, palestras, posts) para o autor
3. Servir como material de venda/consultoria — o conjunto de notas é o argumento de autoridade

## 2. Objetivo

Produzir **23 notas atômicas + 1 MOC** (24 arquivos no total) em `IA/Memória de Agentes/`, todas publicadas (`publish: true`), em PT-BR, cobrindo do conceito ao guia de implementação, com fundamentação acadêmica, comparativo crítico e nota explícita sobre modelo de negócio.

A trilha precisa ser:

- **Atômica** — cada nota linkável e citável separadamente
- **Híbrida em camadas** — abertura acessível, aprofundamento técnico no corpo
- **Rigorosa** — citações a papers e benchmarks reais, não apenas blogs virais
- **Crítica** — desconstrói claims de marketing (ex: "MemPalace 96.6%") com análise de literatura primária
- **Atualizada para abril/2026** — estado da arte real, não material de 2024

## 3. Escopo

### Em escopo

- 24 arquivos markdown na pasta `IA/Memória de Agentes/`
- Todos com `publish: true` (Quartz vai publicar no site público josenaldo.github.io)
- Idioma: PT-BR exclusivamente nesta fase
- Pesquisa abrangente (Abordagem C aprovada): leitura de papers acadêmicos primários, análise de código dos repos GitHub onde necessário, busca complementar para preencher lacunas

### Fora de escopo (trabalho futuro)

- **Publicação bilíngue** (PT-BR + EN). A versão EN será derivada destas notas posteriormente, em projeto separado, depois que o autor decidir qual abordagem adotar de fato.
- **Implementação efetiva** no workflow pessoal do autor. Esta trilha é pesquisa; a decisão sobre qual ferramenta usar (basic-memory? graphify? construir do zero?) é deliberadamente adiada.
- **Productização** (curso pago, consultoria, produto digital). A nota 23 cobre o modelo conceitual; a execução vem depois.
- **Notas sobre os dois links descartados do Inbox**: `Mattbusel/srfm-lab` (lab de trading quantitativo, não memória) e `forrestchang/andrej-karpathy-skills` (princípios de coding, não memória). Ambos serão apenas mencionados em "Veja também" do MOC com explicação do porquê estão fora.

## 4. Audiência e barra de qualidade

**Audiência primária:** dev/knowledge worker que conhece Obsidian e tem noção básica de LLM (sabe o que é uma API de modelo, ouviu falar de RAG), mas que **não** acompanhou o campo de memória de agentes.

**Audiência secundária:** dev sênior que já implementou RAG e quer entender o que muda com agentic memory.

**Barra de qualidade:** ao terminar de ler a trilha, o leitor deve conseguir:

1. Explicar o LLM Wiki Pattern para um cliente não-técnico em 5 minutos
2. Listar 3-5 implementações reais e dizer qual escolher para qual caso
3. Citar pelo menos 2 papers acadêmicos por nome
4. Reconhecer claims de marketing inflados (ex: "100% no LongMemEval em modo híbrido") e responder com análise crítica
5. Implementar uma versão mínima funcional seguindo o guia da nota 22

## 5. Estrutura da trilha (24 arquivos)

### MOC

| # | Arquivo | Propósito |
|---|---------|-----------|
| — | `Memória de Agentes.md` | MOC central. Trilha sequencial recomendada + 4 rotas alternativas (gerencial / técnica / acadêmica / implementador). Inclui callout de aviso sobre o domínio impostor `mempalace.tech` e nota sobre os 2 links descartados do Inbox. |

### Fundamentos (5 notas)

| # | Arquivo | Propósito | Conteúdo-chave |
|---|---------|-----------|----------------|
| 01 | `01 - O que é memória em IA.md` | Introdução acessível ao conceito | Memória in-context vs persistente; analogia com memória humana; por que LLMs "esquecem"; primeiros exemplos práticos |
| 02 | `02 - O problema das janelas de contexto.md` | O problema técnico que motiva tudo | Janelas finitas; custo de tokens; latência; "lost in the middle"; degradação de qualidade em contextos longos; números atualizados (Claude 4.7 1M, GPT-5, etc) |
| 03 | `03 - Taxonomia da memória (episódica, semântica, procedural).md` | Vocabulário fundamental | Distinções de Tulving + adaptações para IA; episódica/semântica/procedural; working memory vs long-term; Park et al. (memory stream); referência a [[17 - Generative Agents]] |
| 04 | `04 - RAG vs memória de longo prazo.md` | Diferenciação conceitual | RAG = retrieval reativo de docs estáticos; memória = construção ativa de representação; quando RAG basta, quando não basta; analogia "biblioteca vs caderno" |
| 05 | `05 - Beyond RAG - quando RAG não basta.md` | A limitação que o LLM Wiki tenta resolver | Multi-session continuity; conhecimento que evolui; conexões implícitas; meta-conhecimento (o que sei sobre o que sei); referência a [[06 - O LLM Wiki Pattern]] |

### O insight do Karpathy (3 notas)

| # | Arquivo | Propósito | Conteúdo-chave |
|---|---------|-----------|----------------|
| 06 | `06 - O LLM Wiki Pattern (gist do Karpathy).md` | Coração da trilha — o insight original | Tweet de 3/abril/2026; gist oficial; arquitetura 3-layer (Raw / Wiki / Schema); operações Ingest/Query/Lint; o esquema da wiki real do Karpathy (~100 artigos, 400k palavras); compiler analogy |
| 07 | `07 - Por que Obsidian e markdown como substrato.md` | Justificativa técnica do substrato | Markdown como formato textual durável; wikilinks como grafo emergente; vantagens vs vector DB puro; portabilidade; Obsidian como "IDE" da wiki |
| 08 | `08 - Arquitetura de um sistema de memória.md` | Generalização da arquitetura | Componentes: ingestão, indexação, retrieval, escrita, lint, governance; diagramas Mermaid; padrões "Storage / Reflection / Experience" do survey 2026 |

### Implementações (8 notas)

| # | Arquivo | Propósito | Conteúdo-chave |
|---|---------|-----------|----------------|
| 09 | `09 - Panorama de implementações (abril 2026).md` | Visão geral comparativa | Mapa do mercado em abril/2026; categorização (Karpathy-inspired / production framework / academic); tabela síntese; entrada para as notas 10-16 |
| 10 | `10 - LLM-knowledge-base (Wendel) — direto do gist.md` | Implementação de referência do gist | Arquitetura kb/ package; 4-stage cycle (ingest/compile/answer/lint); hybrid search (BM25 + RRF); claims lifecycle (confidence, supersession, decay); EPUB/PDF/OCR; integração Obsidian; 311 testes |
| 11 | `11 - graphify — knowledge graph de raw.md` | Versão graph-based | 3-pass processing (AST + audio/video + semantic); NetworkX + Leiden community detection; 25 linguagens via tree-sitter; integração Claude Code/Cursor; "71.5x fewer tokens"; output graph.html + graph.json + GRAPH_REPORT.md |
| 12 | `12 - basic-memory — MCP nativo Obsidian.md` | A opção mais elegante para integração Obsidian | MCP server; mesmo arquivo .md serve LLM e Obsidian; SQLite local index; semantic graph traversável; Docker image disponível; comparativo com soluções não-MCP |
| 13 | `13 - Letta (ex-MemGPT).md` | Sucessor MemGPT, framework de produção | OS analogy (RAM/disk); virtual context management; self-editing memory; tiers (core / archival / message buffer); $10M raised; ausência de score LongMemEval (analisar) |
| 14 | `14 - Mem0 — vetorial + grafo.md` | Framework de produção mais maduro comercialmente | Base Mem0 (vetorial) + Mem0g (graph); 21 frameworks suportados Python+TS; 93.4% LongMemEval (abril/2026); 91% lower latency; arxiv 2504.19413 |
| 15 | `15 - Zep e Graphiti — knowledge graph temporal.md` | Solução KG temporal | Graphiti = engine open-source; Zep Cloud = comercial; Neo4j-based; bi-temporal model; +18.5% LongMemEval; tokens 115k → 1.6k, latência 30s → 3s; arxiv 2501.13956 |
| 16 | `16 - MemPalace (Milla Jovovich).md` | A virada de abril/2026 | Memory palace architecture (wings/rooms/drawers); AAAK 30x compression; 170-token startup; 96.6% LongMemEval; 29 MCP tools; SQLite + grafo temporal; **callout de aviso sobre `mempalace.tech` impostor**; análise honesta com referência a [[21 - Críticas, limitações e armadilhas]] |

### Fundamentação acadêmica (3 notas — review notes)

| # | Arquivo | Propósito | Conteúdo-chave |
|---|---------|-----------|----------------|
| 17 | `17 - Generative Agents (Park, Stanford 2023).md` | Paper foundational | Park et al. UIST 2023; arxiv 2304.03442; memory stream + reflection trees + planning; o sandbox de 25 agentes "Sims"; por que esse paper criou o vocabulário do campo |
| 18 | `18 - A-MEM — Zettelkasten dinâmico.md` | Inovação acadêmica recente | Wujiang Xu et al., NeurIPS 2025; arxiv 2502.12110; nota com atributos estruturados; ligação dinâmica baseada em similaridade; memory evolution (notas antigas se atualizam); inspiração explícita em Luhmann |
| 19 | `19 - Surveys e estado da arte 2026.md` | Mapa do campo | "Memory in the Age of AI Agents: A Survey" (Shichun-Liu); "Memory for Autonomous LLM Agents" (arxiv 2603.07670); "From Storage to Experience" (OpenReview); ICLR 2026 Workshop "MemAgents"; 5 mecanismos (context-resident compression / retrieval-augmented stores / reflective self-improvement / hierarchical virtual context / policy-learned management) |

### Análise crítica e aplicação (4 notas)

| # | Arquivo | Propósito | Conteúdo-chave |
|---|---------|-----------|----------------|
| 20 | `20 - Comparativo crítico (LongMemEval).md` | Tabela comparativa rigorosa | Resultados reais abril/2026 com fontes; quem **não** publicou (Letta, Cognee, LangMem); arquitetura vs performance; recomendações por caso de uso (consultor solo / startup / enterprise / pesquisador) |
| 21 | `21 - Críticas, limitações e armadilhas.md` | A nota que diferencia da hype | Paper crítico arxiv 2604.21284 sobre MemPalace (spatial metaphor não faz o trabalho real); overfitting de benchmark; viés de auto-publicação; quando NÃO usar memória (tarefas one-shot, dados sensíveis, baixo orçamento); custo computacional escondido; "context pollution" |
| 22 | `22 - Guia de implementação do zero.md` | O guia how-to | Setup mínimo do LLM Wiki Pattern em Obsidian + Claude Code; estrutura de pastas; CLAUDE.md schema; primeira ingestão; primeira query; primeiro lint; alternativa pronta (basic-memory MCP); critérios de quando ir além do mínimo |
| 23 | `23 - Aplicações comerciais e modelo de negócio.md` | A nota explicitamente comercial | Personas de cliente; ofertas (consultoria de implementação / setup pronto / treinamento); precificação observada no mercado abril/2026; objeções comuns; ROI mensurável; positioning vs ferramentas SaaS prontas (Mem0, Letta) |

## 6. Convenções aplicadas a todas as notas

### Frontmatter padrão

```yaml
---
title: "<título da nota sem prefixo numérico>"
created: 2026-04-25
updated: 2026-04-25
type: concept   # ou: review (papers), moc (índice)
status: seedling
publish: true
tags:
  - memoria-agentes
  - ia
  - <tags específicas: rag, knowledge-graph, mcp, etc>
aliases:
  - <nomes alternativos para autocomplete de wikilinks>
---
```

### Estrutura interna padrão (para `type: concept`)

```markdown
# <Título>

> [!abstract] TL;DR
> 3-5 linhas que um leigo entende. Esta é a "versão executiva" extraível para posts.

## O que é

Definição clara e objetiva. 1-3 parágrafos.

## Por que importa

O problema que isso resolve, valor concreto. Esta seção é o gancho de venda.

## Como funciona

Aprofundamento técnico. Diagramas Mermaid quando aplicável. Esta é a "versão técnica".

## Quando usar / quando não usar

Critérios de aplicação. Casos limite.

## Armadilhas comuns

Erros frequentes, mal-entendidos, hype vs realidade.

## Veja também

- [[Nota relacionada 1]]
- [[Nota relacionada 2]]

## Referências

- [Paper/post 1](url) — anotação curta
- [Paper/post 2](url) — anotação curta
```

### Estrutura para `type: review` (papers — notas 17, 18)

```markdown
# <Nome do paper>

> [!abstract] TL;DR
> O que o paper propõe e por que é importante para a trilha.

## Metadados

- **Autores:** ...
- **Venue:** ...
- **Ano:** ...
- **arXiv / DOI:** ...
- **Código:** ...

## Problema

O que o paper tenta resolver.

## Contribuição

A inovação técnica em si.

## Como funciona

Explicação da arquitetura/método com Mermaid se aplicável.

## Resultados

Métricas e benchmarks reportados, com leitura crítica.

## Limitações reconhecidas pelos autores

## Crítica externa

Recepção da comunidade, follow-ups, paper de crítica se houver.

## Por que importa para a trilha

Conexão com [[O LLM Wiki Pattern]], implementações que se inspiram nele, etc.

## Veja também
## Referências
```

### Recursos visuais e formatação

- **Wikilinks** entre notas. Usar formato `[[NN - Título da nota|texto opcional]]`
- **Callouts:**
  - `> [!tip]` — recomendações práticas
  - `> [!warning]` — armadilhas e limitações importantes
  - `> [!info]` — fatos contextuais
  - `> [!example]` — exemplos concretos
  - `> [!quote]` — citações diretas (Karpathy, papers, etc)
- **Mermaid** nas notas 06, 08, 11, 13, 14, 15, 16, 22 (qualquer lugar com arquitetura)
- **Tabelas** comparativas nas notas 09 e 20

## 7. MOC — `Memória de Agentes.md`

```markdown
---
title: "Memória de Agentes"
type: moc
publish: true
tags: [memoria-agentes, ia, moc]
---

# Memória de Agentes (LLM Wiki Pattern)

[abertura: 3-4 frases sobre o tema, do que se trata, por que importa em 2026]

## Comece por aqui

[trilha sequencial recomendada — link para 01 → 23 com 1 linha cada]

## Rotas alternativas

### Rota gerencial (entender e vender)
01 → 02 → 06 → 09 → 23

### Rota técnica (implementar)
04 → 06 → 08 → 09 → 22

### Rota acadêmica (fundamentar discurso)
03 → 17 → 18 → 19 → 21

### Rota implementador (mão na massa rápida)
06 → 12 → 22

## Notas

[lista completa das 23 notas com 1 linha de descrição cada]

> [!warning] Avisos importantes
> - O domínio `mempalace.tech` é impostor com malware. Apenas `github.com/milla-jovovich/mempalace` e `github.com/MemPalace/mempalace` são oficiais.
> - Dois links que apareceram nas pesquisas iniciais NÃO são sobre memória de agentes apesar do nome: `Mattbusel/srfm-lab` (lab de trading quantitativo) e `forrestchang/andrej-karpathy-skills` (princípios de coding do Karpathy). Eles ficam fora desta trilha.

```dataview
LIST
FROM "IA/Memória de Agentes"
WHERE type != "moc"
SORT file.name ASC
```
```

## 8. Bibliografia anotada (fontes-âncora)

### Fontes primárias

- **Karpathy gist** — `gist.github.com/karpathy/442a6bf555914893e9891c11519de94f` — o documento original do LLM Wiki Pattern. Fonte primária para nota 06.
- **Tweet do Karpathy** (3/abril/2026) — `x.com/karpathy/status/2040470801506541998` — anúncio público. Citar diretamente.

### Papers acadêmicos (fontes primárias)

- **Generative Agents** — Park et al., UIST 2023 — `arxiv.org/abs/2304.03442` — base para nota 17, citada em 03, 19
- **MemGPT** — Packer et al., 2023 — `arxiv.org/abs/2310.08560` — base para nota 13, citada em 19
- **A-MEM** — Wujiang Xu et al., NeurIPS 2025 — `arxiv.org/abs/2502.12110` — base para nota 18
- **Mem0** — 2025 — `arxiv.org/abs/2504.19413` — base para nota 14
- **Zep** — 2025 — `arxiv.org/abs/2501.13956` — base para nota 15
- **Memory for Autonomous LLM Agents (Survey)** — 2026 — `arxiv.org/abs/2603.07670` — base para nota 19
- **Spatial Metaphors for LLM Memory: A Critical Analysis of MemPalace** — 2026 — `arxiv.org/abs/2604.21284` — base para nota 21, citado em 16, 20

### Repositórios (análise de código)

- `github.com/karpathy/442a6bf555914893e9891c11519de94f` (gist)
- `github.com/wendeus0/LLM-knowledge-base` — nota 10
- `github.com/safishamsi/graphify` — nota 11
- `github.com/basicmachines-co/basic-memory` — nota 12
- `github.com/letta-ai/letta` — nota 13
- `github.com/mem0ai/mem0` — nota 14
- `github.com/getzep/graphiti` — nota 15
- `github.com/milla-jovovich/mempalace` — nota 16
- `github.com/joonspk-research/generative_agents` — nota 17
- `github.com/agiresearch/A-mem` ou `github.com/WujiangXu/A-mem` — nota 18
- `github.com/xiaowu0162/LongMemEval` — nota 20
- `github.com/Shichun-Liu/Agent-Memory-Paper-List` — nota 19

### Editorial de qualidade (secundárias)

- VentureBeat: "Karpathy shares 'LLM Knowledge Base' architecture that bypasses RAG"
- Level Up Coding (Plaban Nayak): "Beyond RAG: How Andrej Karpathy's LLM Wiki Pattern Builds Knowledge That Actually Compounds"
- The New Stack: "Memory for AI Agents: A New Paradigm of Context Engineering"
- LangChain blog: "Context Engineering for Agents"
- Mem0 official: "State of AI Agent Memory 2026"
- MindStudio: "Karpathy's LLM Knowledge Base Architecture: The Compiler Analogy"
- HPCwire/BigDATAwire: "Letta Emerges from Stealth with $10M"

### Implementações inspiradas no Karpathy (mencionadas em 09 e 22)

- `github.com/NicholasSpisak/second-brain`
- Apify "Second Brain Builder" (`apify.com/openclawai/second-brain-builder`)
- Gist `rohitg00/2067ab416f7bbe447c1977edaaa681e2` (LLM Wiki v2)

## 9. Critérios de qualidade por nota

Antes de marcar uma nota como concluída (`status: evergreen`), verificar:

- [ ] **TL;DR** existe e é compreensível para um leigo em <30 segundos
- [ ] **3+ fontes citadas** (mínimo: 1 primária, 1 acadêmica ou repo, 1 editorial)
- [ ] **Nenhuma alegação técnica não suportada** por fonte
- [ ] **Pelo menos 2 wikilinks** para outras notas da trilha
- [ ] **Pelo menos 1 callout** (`> [!warning]`, `> [!tip]`, etc)
- [ ] **Mermaid** se a nota descreve arquitetura
- [ ] **Frontmatter completo** (tags, aliases se aplicável, type correto)
- [ ] **Seção "Quando NÃO usar"** ou equivalente (evita postura de evangelista)
- [ ] **Português brasileiro natural** (nada de tradução literal de inglês)
- [ ] **Termos técnicos em inglês mantidos** quando consagrados (RAG, embedding, LongMemEval, Zettelkasten, etc) — nada de "geração aumentada por recuperação"

## 10. Plano de execução (high-level)

A escrita das notas tem dependências (algumas precisam existir antes de outras serem citadas). Sequência sugerida (detalhamento fica para o `writing-plans` skill):

1. **Wave 1 — Esqueleto:** MOC + 06 (LLM Wiki Pattern) — definem o vocabulário do resto
2. **Wave 2 — Fundamentos:** 01, 02, 03, 04, 05 — base conceitual
3. **Wave 3 — Karpathy core:** 07, 08 — derivam de 06
4. **Wave 4 — Acadêmico:** 17, 18, 19 — review notes, podem ser feitas em paralelo
5. **Wave 5 — Implementações:** 09 (panorama), depois 10-16 em paralelo
6. **Wave 6 — Síntese:** 20 (comparativo) e 21 (críticas) — dependem de todas as anteriores
7. **Wave 7 — Aplicação:** 22 (guia) e 23 (negócio) — fechamento
8. **Wave 8 — Revisão e linkagem:** revisar wikilinks, MOC, consistência cruzada

## 11. Trabalho fora de escopo (futuro)

Documentar aqui para não esquecer e para o autor decidir prazos depois:

- **Publicação bilíngue (PT-BR + EN)** — depois desta trilha pronta, quando o autor decidir a direção, fazer versão EN para alcance internacional. Provavelmente exige adaptação de tom e referências, não é tradução pura.
- **Decisão sobre integração no workflow pessoal** — qual ferramenta o autor vai adotar de fato (basic-memory? construir do zero a partir do gist? graphify para o Codex?). Decisão fora desta pesquisa.
- **Productização** — curso pago, consultoria, oferta empacotada. A nota 23 conceitua, mas o produto efetivo é projeto separado.
- **Notas sobre Karpathy adjacentes** — `forrestchang/andrej-karpathy-skills` poderia virar nota em outra parte do vault (não em "Memória de Agentes"); idem materiais sobre Karpathy não relacionados a memória.
- **Demos práticas** — gravar um vídeo curto demonstrando o LLM Wiki em ação no Codex. Material de marketing, fora desta trilha.

## 12. Riscos e mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| Campo se mover rápido (paper novo, ferramenta nova) durante a escrita | Alta | Médio | Datar todas as notas; revisitar em 6 meses; nota 19 (surveys) é o pivô a atualizar |
| Confundir hype de Twitter com técnica madura | Média | Alto | Nota 21 (críticas) é mandatory antes de publicar; usar paper crítico arxiv 2604.21284 |
| Notas longas demais para serem atômicas | Média | Médio | Limite: ~3000 palavras por nota; se ultrapassar, sub-dividir |
| Linkagem inconsistente entre notas | Média | Baixo | Wave 8 (revisão) faz audit; o Quartz mostra backlinks para conferir |
| Tom de "evangelista" (vendendo demais o LLM Wiki) | Baixa | Alto | Critério "Quando NÃO usar" obrigatório por nota; nota 21 explícita |
| Conteúdo do paper crítico (arxiv 2604.21284) ser propaganda disfarçada da MemPalace | Baixa | Médio | Verificar autoria, peer review, citações cruzadas antes de usar como fonte central |

## 13. Definição de "concluído"

A trilha está concluída quando:

1. Os 24 arquivos existem em `IA/Memória de Agentes/`
2. Cada nota passa nos 10 critérios de qualidade da seção 9
3. O MOC tem dataview funcional listando todas as notas
4. Wikilinks estão consistentes (nenhum link quebrado)
5. Quartz publica sem erros
6. O autor consegue, em 30 minutos, percorrer a trilha sequencial e responder aos 5 testes da seção 4
