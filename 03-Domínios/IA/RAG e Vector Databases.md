---
title: "RAG e Vector Databases"
created: 2026-04-11
updated: 2026-04-11
type: concept
status: evergreen
tags:
  - ia
  - rag
  - embeddings
  - entrevista
  - fundamentos
publish: true
---

# RAG e Vector Databases

> Quase toda aplicação séria com LLM em 2026 tem RAG no meio do caminho. O motivo é simples: LLMs conhecem muita coisa mas não conhecem **seus dados** — sua documentação interna, políticas, base de clientes, histórico do paciente. **RAG (Retrieval-Augmented Generation)** é a técnica que injeta dados específicos no contexto do LLM em runtime, sem precisar treinar nada. Esta nota é a trilha do zero ao domínio: embeddings, chunking, vector databases, retrieval, ranking, evaluation, e como NÃO fazer RAG burro — porque a distância entre "RAG de tutorial" e "RAG que funciona em produção" é enorme.

## O que é

**RAG** combina dois componentes:

1. **Retrieval:** dado uma pergunta, busca os trechos mais relevantes em uma base de conhecimento.
2. **Generation:** passa esses trechos como contexto ao LLM, que gera a resposta baseada neles.

O resultado é um LLM que "parece" conhecer seus dados, mas na verdade está apenas lendo os trechos injetados em runtime. É barato (comparado a fine-tuning), flexível (atualizar base = atualizar documentos), e dá a capacidade chave: **citar fontes**.

```text
[User pergunta] → [Retrieval] → [trechos relevantes] ┐
                                                     ▼
                                [LLM com contexto] → [Resposta citando fontes]
```

## O que diferencia um senior em RAG

1. **Sabe que RAG não é sobre vector database, é sobre retrieval quality.** Vector DB é commodity; chunking, hybrid search e reranking são onde a qualidade vive.
2. **Nunca usa pure vector search em produção.** Hybrid (BM25 + vector) com reranker é o padrão.
3. **Trata chunking com seriedade.** Chunks ruins = RAG ruim, sem recuperação.
4. **Mede retrieval quality separadamente de generation quality.** Se retrieval falha, generation não salva.
5. **Conhece as armadilhas:** tabela de conteúdos em vez de conteúdo, chunks sem metadata, queries mal formuladas, ranking ingênuo.
6. **Implementa query rewriting.** Pergunta do usuário raramente é a melhor query.
7. **Tem evaluation de RAG:** faithfulness, relevance, context precision/recall.
8. **Sabe quando RAG ≠ resposta** — e devolve "não sei" ou "o contexto não cobre isso".
9. **Faz tiering:** se o contexto é pequeno e estável, joga tudo no prompt. RAG só quando necessário.
10. **Não confunde RAG com fine-tuning.** Sabe escolher quando cada um faz sentido.

## Trilha de aprendizado — do zero ao domínio

### Nível 1 — Conceito (3-5 dias)

- Entenda o loop: embed docs → store → embed query → retrieve top-k → generate.
- Embeddings como representação vetorial, similaridade de cosseno.
- Leia: [Pinecone — Learn RAG](https://www.pinecone.io/learn/retrieval-augmented-generation/).

**Check:** você explica RAG para um colega em 5 minutos com um diagrama.

### Nível 2 — Construir um RAG mínimo (1-2 semanas)

- Stack simples: SQLite + pgvector (ou Pinecone free tier).
- Indexe 100 documentos, faça Q&A.
- Chunking fixo de ~500 tokens.
- Top-k retrieval por similarity.
- Ver onde falha.

**Check:** você tem um RAG funcionando num dataset próprio.

### Nível 3 — Retrieval de qualidade (2-4 semanas)

- Hybrid search (BM25 + vector).
- Reranking (Cohere Rerank, Voyage Rerank).
- Chunking semântico / hierárquico.
- Query rewriting e HyDE.
- Metadata filtering.

**Check:** você consegue justificar cada escolha em cima de métricas, não "parece melhor".

### Nível 4 — Evaluation e produção (2-4 semanas)

- Golden set com (pergunta, resposta correta, contexto esperado).
- Métricas: context precision, context recall, faithfulness, answer relevance.
- Ragas / TruLens / DeepEval.
- Observabilidade via Langfuse.
- Cost e latency profiling.

**Check:** você tem um RAG em produção com métricas monitoradas.

### Nível 5 — Avançado (ongoing)

- Multi-hop retrieval (responder perguntas que precisam juntar múltiplos documentos).
- Graph RAG (Microsoft GraphRAG, Neo4j).
- Agentic RAG (agent decide quando/o que buscar).
- Modelos de embedding domain-specific.
- Fine-tuning de embeddings.

## Anatomia de um pipeline RAG

```text
INDEXING (offline, uma vez)
───────────────────────────
Documentos originais
   │
   ▼
┌─────────────┐
│ 1. Parse    │  PDF, HTML, MD → texto estruturado
└─────────────┘
   │
   ▼
┌─────────────┐
│ 2. Chunk    │  texto → pedaços de N tokens com overlap
└─────────────┘
   │
   ▼
┌─────────────┐
│ 3. Embed    │  cada chunk → vetor denso (ex: 1536 dims)
└─────────────┘
   │
   ▼
┌─────────────┐
│ 4. Store    │  vector DB: (chunk_text, embedding, metadata)
└─────────────┘

QUERY (online, cada pergunta)
─────────────────────────────
User question
   │
   ▼
┌─────────────┐
│ 5. Rewrite  │  transformar pergunta em query melhor (opcional)
└─────────────┘
   │
   ▼
┌─────────────┐
│ 6. Embed    │  query → vetor
└─────────────┘
   │
   ▼
┌─────────────┐
│ 7. Retrieve │  similarity search (vector) + BM25 (keyword)
└─────────────┘
   │
   ▼
┌─────────────┐
│ 8. Rerank   │  reranker seleciona top-k finais
└─────────────┘
   │
   ▼
┌─────────────┐
│ 9. Generate │  LLM recebe query + chunks, produz resposta
└─────────────┘
   │
   ▼
Answer with citations
```

Cada passo é uma oportunidade para melhorar (ou destruir) a qualidade.

## Embeddings — a representação semântica

**Embedding** é a representação vetorial de texto (ou outro dado) como array de números, tipicamente 256-3072 dimensões. Textos com significado similar ficam próximos nesse espaço.

### Como se mede similaridade

**Cosine similarity** (padrão): mede o ângulo entre dois vetores, resultado entre -1 e 1.

```text
cos(v1, v2) = (v1 · v2) / (||v1|| × ||v2||)
```

Cosseno 1.0 = mesmos, 0 = ortogonais, -1 = opostos. Para embeddings modernos, tipicamente 0.7+ é "relacionado".

**Dot product:** similar, sem normalizar. Mais rápido se vetores já são normalizados (o que os modelos de embedding fazem).

**Euclidean distance:** menos comum, prefere-se cosseno.

### Modelos de embedding em 2026

| Modelo                 | Provedor | Dims | Qualidade       | Preço            |
| ---------------------- | -------- | ---- | --------------- | ---------------- |
| text-embedding-3-large | OpenAI   | 3072 | Alta            | $$               |
| text-embedding-3-small | OpenAI   | 1536 | Boa             | $                |
| voyage-3               | Voyage   | 1024 | Muito alta      | $$$              |
| voyage-3-lite          | Voyage   | 512  | Boa             | $                |
| embed-v3               | Cohere   | 1024 | Muito alta      | $$               |
| nomic-embed-text       | Nomic    | 768  | Boa, open       | grátis self-host |
| bge-m3                 | BAAI     | 1024 | Muito boa, open | grátis self-host |

**Em 2026**, Voyage e Cohere consistentemente batem OpenAI em benchmarks de retrieval. BAAI bge-m3 é o melhor open source. Para domínios específicos (legal, medical, code), modelos especializados existem.

### Matryoshka embeddings

Modelos modernos (OpenAI v3, Voyage 3) produzem embeddings "matryoshka": você pode truncar para menos dimensões (ex: 1536 → 512) mantendo boa qualidade. Útil para reduzir custo de armazenamento.

### Escolha do modelo

- **Comece com text-embedding-3-small ou voyage-3-lite.** Custo-benefício ótimo.
- **Para domínio específico**, avalie Voyage (oferece finetunes domain-specific) ou modelo local.
- **Para máxima qualidade**, voyage-3 ou embed-v3.
- **Para self-host**, bge-m3 ou nomic-embed-text.

**Critério de escolha real:** rode benchmark MTEB para seu dataset. Não confie só em benchmarks públicos.

## Chunking — onde 50% da qualidade vive

Chunking mal feito é o maior destruidor de RAG. Princípios:

### Tamanho

- **Muito pequeno (< 200 tokens):** perde contexto, chunks inúteis isolados.
- **Muito grande (> 1500 tokens):** dilui relevância, embeddings pobres.
- **Sweet spot:** 300-800 tokens, depende do domínio.

### Overlap

**Por quê:** info na borda de um chunk pode ser parcial. Overlap de 10-20% (ex: 100 tokens em chunks de 500) evita perder info que cai exatamente na borda.

### Estratégias de chunking

**Fixed-size (burro):** divide por N caracteres/tokens. Rápido, destrutivo. Quebra no meio de frases, tabelas, exemplos.

**Recursive character splitter:** divide por hierarquia de separadores (\n\n → \n → . → espaço). Usado por LangChain. Melhor que fixed.

**Semantic chunking:** usa embeddings para encontrar limites semânticos (onde o significado muda). Mais cara mas melhor qualidade.

**Hierarchical chunking:** divide em níveis (documento → seção → parágrafo → frase). Armazena múltiplas granularidades, escolhe dinamicamente.

**Structure-aware chunking:** aproveita estrutura do doc (headers markdown, seções HTML, listas). Respeita limites naturais. **Este é o padrão de ouro para docs técnicos.**

**Late chunking** (novo, 2024): embedda o documento inteiro, depois divide a sequência de embeddings em chunks. Chunks individuais preservam contexto global. Requer modelo com contexto longo.

### Metadata

Cada chunk deve ter metadata útil:

- `source_url`, `title`
- `section`, `subsection`
- `doc_type` (ex: FAQ, manual, API reference)
- `date`, `author`
- `language`
- Qualquer campo útil para filtragem

Metadata permite **filtered retrieval** ("só responda com docs da área X" ou "só docs de 2025").

### Exemplo: chunking markdown estruturado

```python
def chunk_markdown(md: str, max_tokens: int = 600, overlap: int = 100):
    # Quebra em headers
    sections = split_by_headers(md, min_level=2)  # ## e mais profundo
    chunks = []
    for section in sections:
        section_tokens = count_tokens(section.text)
        if section_tokens <= max_tokens:
            chunks.append(Chunk(
                text=section.text,
                metadata={"section": section.title, "level": section.level}
            ))
        else:
            # Subdivide por parágrafos com overlap
            subchunks = split_with_overlap(
                section.text, max_tokens, overlap
            )
            for i, sub in enumerate(subchunks):
                chunks.append(Chunk(
                    text=sub,
                    metadata={
                        "section": section.title,
                        "subchunk": i,
                        "level": section.level
                    }
                ))
    return chunks
```

## Vector databases — onde os embeddings vivem

### O que fazem

Vector DBs armazenam vetores densos e respondem queries "encontre os N mais similares". Por baixo, usam estruturas como **HNSW** (Hierarchical Navigable Small World) para aproximar top-k em O(log n).

### Opções em 2026

| DB                       | Tipo                 | Prós                                     | Contras                                 |
| ------------------------ | -------------------- | ---------------------------------------- | --------------------------------------- |
| **pgvector**             | PostgreSQL extension | Unificado com dados relacionais, simples | Menos otimizado que dedicados em escala |
| **Pinecone**             | SaaS dedicado        | Managed, escala, rápido                  | Custo, lock-in                          |
| **Weaviate**             | Open source + SaaS   | Híbrido BM25+vector built-in, schemas    | Mais complexo                           |
| **Qdrant**               | Open source + SaaS   | Rust, rápido, filtros avançados          | Comunidade menor                        |
| **Milvus**               | Open source + SaaS   | Altíssima escala, otimizado              | Mais pesado operacionalmente            |
| **LanceDB**              | Embedded             | Simples como SQLite, sem servidor        | Menos features                          |
| **ChromaDB**             | Embedded/SaaS        | Simples para dev                         | Menos maduro para produção              |
| **Redis Stack**          | In-memory            | Rápido, já tem Redis                     | Custo de RAM                            |
| **OpenSearch / Elastic** | Search engine        | Hybrid search nativo, maduro             | Overhead de operação                    |

### Como escolher

- **Projeto pequeno / protótipo:** pgvector (se você já tem Postgres) ou LanceDB / ChromaDB.
- **Projeto médio:** pgvector sério (com índice HNSW), Qdrant, Weaviate.
- **Escala (milhões+):** Pinecone, Milvus, Qdrant, Weaviate.
- **Já tem Elastic/OpenSearch:** use, tem hybrid nativo.
- **Stack Postgres-first:** pgvector. Em 2026 é suficientemente maduro para produção séria.

**Meu default:** pgvector. Mantém stack simples, roda bem até a escala onde você já pode pagar Pinecone sem sentir.

### pgvector em 30 segundos

```sql
CREATE EXTENSION vector;

CREATE TABLE chunks (
  id SERIAL PRIMARY KEY,
  doc_id TEXT NOT NULL,
  section TEXT,
  content TEXT NOT NULL,
  embedding vector(1536),  -- OpenAI text-embedding-3-small
  metadata JSONB
);

CREATE INDEX ON chunks USING hnsw (embedding vector_cosine_ops);

-- Query: top-10 mais similares
SELECT id, content, 1 - (embedding <=> query_embedding) AS similarity
FROM chunks
WHERE metadata->>'lang' = 'pt'
ORDER BY embedding <=> query_embedding
LIMIT 10;
```

Note: `<=>` é cosine distance (0 = idêntico, 2 = opostos). `1 - <=>` vira similaridade.

## Retrieval — muito além de "top-k por cosseno"

### O problema do pure vector search

Vector search é excelente para similaridade semântica ("conceitos relacionados") mas **ruim para termos exatos**. Se o usuário procura "erro E-4032", vector search pode retornar chunks sobre erros em geral, não aquele específico. **Keyword search** é crítico para esses casos.

### Hybrid search — o padrão moderno

Combinar **BM25** (keyword) + **vector** (semântico):

1. Rodar ambos em paralelo.
2. Pegar top-N de cada.
3. Fundir resultados usando **RRF** (Reciprocal Rank Fusion) ou pesos.

```python
def hybrid_search(query: str, k: int = 20):
    query_emb = embed(query)
    vec_results = vector_db.search(query_emb, top_k=k)   # top-20 semânticos
    bm25_results = bm25_index.search(query, top_k=k)     # top-20 keyword
    return reciprocal_rank_fusion(vec_results, bm25_results, k=60)

def reciprocal_rank_fusion(list1, list2, k=60):
    scores = {}
    for i, item in enumerate(list1):
        scores[item.id] = scores.get(item.id, 0) + 1 / (k + i)
    for i, item in enumerate(list2):
        scores[item.id] = scores.get(item.id, 0) + 1 / (k + i)
    return sorted(scores.items(), key=lambda x: -x[1])
```

Weaviate, OpenSearch e outros fazem isso built-in. Para pgvector, você combina com `ts_rank` (full-text search do Postgres).

### Reranking — o segundo filtro que muda tudo

Depois de retrieval inicial (top-20 ou top-50), passe por um **reranker** — modelo menor dedicado a ordenar resultados por relevância real, tipicamente cross-encoder.

**Por que funciona:** vector search usa um único embedding da query contra embeddings pré-computados. Reranker **lê a query junto com cada resultado** e pontua cada par. Muito mais preciso, só computacionalmente viável em top-N pequeno.

**Modelos de rerank:**

- Cohere Rerank v3 (API)
- Voyage Rerank
- bge-reranker-v2 (open source)
- Jina Reranker

Impacto real: em benchmarks e projetos, adicionar reranker tipicamente sobe qualidade em 15-30 pontos percentuais. **É a otimização mais subestimada.**

### Query rewriting

A pergunta do usuário raramente é a melhor query. Técnicas:

**Expansion:** LLM gera variações semânticas. "Como exporto CSV?" → [como exporto CSV, exportar para CSV, baixar como planilha, download CSV].

**HyDE (Hypothetical Document Embeddings):** gerar uma "resposta hipotética" com LLM, embedar a resposta, usar o embedding para busca. Funciona porque respostas estão no mesmo "espaço" de documentos.

**Decomposition:** quebrar pergunta complexa em múltiplas sub-perguntas. "Quais são as diferenças entre A e B e quando usar cada um?" → 3 queries.

**Clarification:** em agents, pedir clarificação quando pergunta é ambígua.

### Metadata filtering

Filtrar antes ou durante retrieval por metadata: idioma, data, autor, tipo de doc, tenant. **Obrigatório** em multi-tenant.

```sql
-- Exemplo pgvector com filtro
SELECT * FROM chunks
WHERE metadata->>'tenant_id' = $1
  AND (metadata->>'date')::date >= '2024-01-01'
ORDER BY embedding <=> $2
LIMIT 10;
```

### Multi-vector retrieval

Para documentos longos: embedar múltiplos pedaços (summary, chunks, questions-this-answers) e recuperar documento por qualquer um. Aumenta recall.

## Generation — como passar contexto ao LLM

### Prompt template

```text
You are an assistant answering questions based on the provided context.

<context>
{retrieved_chunks_formatted}
</context>

Rules:
- Answer ONLY based on the context above.
- If the context does not contain the answer, say "I don't have information about that in my sources."
- Cite sources using [source: title] at the end of each relevant sentence.
- Be concise.

Question: {user_question}
```

**Chaves:**

- **Delimitação XML** do contexto.
- **Instrução explícita** de responder só baseado no contexto.
- **Fallback** quando não há info.
- **Citação de fonte** obrigatória.

### Formatação de chunks no prompt

```text
<context>
[Source 1: "Installation Guide", section "Requirements"]
The system requires Python 3.11 or higher...

[Source 2: "FAQ", section "Common errors"]
If you see E-4032, it means the config file is malformed...
</context>
```

Cada chunk com identificação clara + metadata relevante. Permite citação precisa.

### Cuidado com "lost in the middle"

Com muitos chunks (> 10), info no meio é usada pior. Mitigações:

- Use reranker para garantir que os melhores estão no topo.
- Coloque os top-3 no início e no fim.
- Limite chunks a 5-7 em prompts típicos.

## Evaluation de RAG

### Métricas fundamentais

**Retrieval metrics** (quão bem você recupera):

- **Context precision:** dos chunks recuperados, quantos são relevantes?
- **Context recall:** dos chunks relevantes que existiam, quantos foram recuperados?
- **MRR (Mean Reciprocal Rank):** em que posição o primeiro chunk relevante aparece?
- **nDCG:** qualidade considerando posição.

**Generation metrics** (quão bem o LLM responde):

- **Faithfulness:** a resposta é suportada pelo contexto? (não alucina além do que foi dado)
- **Answer relevance:** a resposta responde a pergunta?
- **Answer correctness:** a resposta está factualmente correta?

### Golden set para RAG

Crie 30-100 tuplas: `(pergunta, resposta_correta, chunks_esperados)`. Rode a cada mudança.

### Frameworks de eval

- **Ragas** — o mais popular para eval de RAG.
- **TruLens** — similar, com UI.
- **DeepEval** — unit testing style.
- **LangSmith** — datasets + eval integrados com LangChain.
- **Langfuse** — observabilidade + eval.

### Eval de retrieval separada de generation

Crítico: se retrieval é ruim, generation não conserta. Meça os dois separados.

1. Rodar retrieval, comparar chunks recuperados com esperados.
2. Rodar generation com chunks **dos esperados** (oráculo) — isso mede só generation.
3. Rodar pipeline completo.

Se (3) piorou mas (2) está ok, seu retrieval é o gargalo.

## RAG vs long context vs fine-tuning

Decisão comum:

### RAG vs long context

Com modelos de 1M+ tokens, surge a pergunta: "por que não jogar tudo no contexto?"

**Long context ganha quando:**

- Base é pequena (< 500K tokens) e estável.
- Você precisa de raciocínio cruzado entre muitos docs.
- Custo não é sensível.

**RAG ganha quando:**

- Base é grande (milhões de tokens).
- Muda frequentemente.
- Custo/latência importam.
- Você precisa de citação precisa.
- Você precisa de controle fino (filtros, permissões).

**"Lost in the middle"** favorece RAG para bases grandes — mesmo com 1M tokens, modelos perdem info no meio.

### RAG vs fine-tuning

**Fine-tuning ganha para:**

- Mudar formato/estilo de saída consistentemente.
- Incorporar vocabulário muito específico.
- Reduzir tokens (modelo "já sabe" instruções recorrentes).

**RAG ganha para:**

- Conhecimento que muda.
- Necessidade de citar fontes.
- Atualizações frequentes.
- Controle fino.

**Regra prática:** comece com RAG. Considere fine-tuning só se RAG falhou ou se padrão é claramente "comportamento", não "conhecimento".

## Armadilhas comuns

### 1. Pure vector search em produção

Sintoma: queries com termos específicos retornam chunks irrelevantes. **Fix:** hybrid search.

### 2. Chunking ingênuo

Chunks quebrados no meio de frases/tabelas. **Fix:** structure-aware chunking.

### 3. Sem reranker

Retrieval rank ruim = generation ruim. **Fix:** adicionar reranker em cima de top-20.

### 4. Chunks sem metadata

Impossível filtrar por tenant/idioma/data. **Fix:** metadata desde o início.

### 5. Query = pergunta do usuário direto

Pergunta conversacional ≠ query ótima. **Fix:** query rewriting ou HyDE.

### 6. LLM alucina fora do contexto

Prompt permite o modelo "adivinhar". **Fix:** instrução explícita "responda só baseado no contexto", "diga 'não sei' quando não tiver info".

### 7. Contexto excessivo

50 chunks no prompt, perde qualidade. **Fix:** top-5 a top-7 reranked.

### 8. Sem evaluation

"Parece funcionar." **Fix:** Ragas, golden set, métricas.

### 9. Mesma strategia para todos os tipos de query

Uma query factual precisa de coisa diferente de query de "como fazer". **Fix:** query classification + estratégias específicas.

### 10. Esquecer custo de embedding

Re-embedar toda a base sem necessidade custa caro. **Fix:** pipeline incremental, cache de embeddings.

### 11. Sem cache de embedding da query

Usuários fazem perguntas similares. **Fix:** cache (LRU ou Redis) pelos últimos N queries.

### 12. Metadata sem índice

Filtros lentos arruínam latência. **Fix:** índices em campos filtráveis.

## Exemplo prático — RAG mínimo funcional

```python
from anthropic import Anthropic
from openai import OpenAI
import psycopg
import numpy as np

anthropic = Anthropic()
openai = OpenAI()
db = psycopg.connect("postgresql://localhost/ragdemo")

def embed(text: str) -> list[float]:
    resp = openai.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return resp.data[0].embedding

def index_document(doc_id: str, title: str, content: str):
    chunks = chunk_markdown(content, max_tokens=600, overlap=100)
    for i, chunk in enumerate(chunks):
        emb = embed(chunk.text)
        with db.cursor() as cur:
            cur.execute(
                """INSERT INTO chunks (doc_id, chunk_index, title, content, embedding, metadata)
                   VALUES (%s, %s, %s, %s, %s, %s)""",
                (doc_id, i, title, chunk.text, emb, json.dumps(chunk.metadata))
            )
    db.commit()

def retrieve(query: str, k: int = 5) -> list[dict]:
    query_emb = embed(query)
    with db.cursor() as cur:
        cur.execute(
            """SELECT doc_id, title, content, 1 - (embedding <=> %s::vector) AS score
               FROM chunks
               ORDER BY embedding <=> %s::vector
               LIMIT %s""",
            (query_emb, query_emb, k)
        )
        return [
            {"doc_id": row[0], "title": row[1], "content": row[2], "score": row[3]}
            for row in cur.fetchall()
        ]

def answer(question: str) -> dict:
    chunks = retrieve(question, k=5)
    if not chunks or chunks[0]["score"] < 0.5:
        return {"answer": "Não encontrei informações relevantes.", "sources": []}

    context = "\n\n".join(
        f"[Source: {c['title']}]\n{c['content']}"
        for c in chunks
    )

    system = """You are a helpful assistant that answers based ONLY on the provided context.
If the context does not contain the answer, say you don't know.
Always cite sources using [source: title]."""

    user = f"""<context>
{context}
</context>

Question: {question}"""

    response = anthropic.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=system,
        messages=[{"role": "user", "content": user}]
    )

    return {
        "answer": response.content[0].text,
        "sources": [{"title": c["title"], "doc_id": c["doc_id"]} for c in chunks]
    }
```

Esse é um RAG funcional. Para melhorar de verdade:

1. Adicionar BM25 e hybrid search.
2. Adicionar reranker em cima dos top-20.
3. Query rewriting.
4. Ragas para evaluation.
5. Prompt caching no system prompt.

## Graph RAG (avançado)

**Problema que resolve:** perguntas que exigem **juntar informação de múltiplos documentos** ou raciocínio sobre relações ("quais pacientes do Dr. X também foram atendidos pela Dra. Y e tiveram diagnóstico Z?").

**Como funciona:**

1. Extrai entidades e relações dos documentos usando LLM.
2. Constrói grafo de conhecimento (Neo4j, NetworkX).
3. Em query, combina retrieval tradicional + travessia de grafo.

**Referências:**

- [Microsoft GraphRAG](https://microsoft.github.io/graphrag/)
- [LlamaIndex Knowledge Graph](https://docs.llamaindex.ai/en/stable/examples/index_structs/knowledge_graph/)

**Quando usar:** multi-hop reasoning, domain com entidades ricas (medicina, legal, supply chain). Complexo e caro — só quando justificado.

## Agentic RAG

RAG onde um **agent** decide o que buscar, quando buscar de novo, e quando parar. Em vez de pipeline fixo.

**Vantagens:**

- Pode expandir busca se primeira não trouxe suficiente.
- Pode decompor pergunta complexa em sub-queries.
- Pode usar múltiplas fontes.

**Desvantagens:**

- Mais caro (mais LLM calls).
- Mais imprevisível.
- Harder to debug.

Ver [[Agents]] para o conceito geral.

## Como ganhar experiência prática

Esta nota é conhecimento estruturado sobre RAG, mas conhecimento sem prática não destrava entendimento profundo nem credibilidade técnica. Três caminhos curados, do menor para o maior esforço:

### Caminho 1 — RAG mínimo em 1-2 dias (validar o conceito)

Fazer o **Lab 1** da seção "Exercícios hands-on" abaixo: pgvector + OpenAI embeddings + Claude Sonnet sobre ~50 documentos próprios, golden set de 20 perguntas, Ragas para medir. Stack mínima, foco em rodar end-to-end uma vez.

**Critério de sucesso:** responder perguntas sobre os docs com citação de fonte; identificar concretamente onde retrieval falha.

### Caminho 2 — RAG sobre o próprio Codex Technomanticus (1-2 semanas)

Indexar este próprio vault (~100 notas em PT-BR e crescendo) e expor via interface conversacional. Força lidar com:

- Chunking de markdown estruturado (headers, callouts, code blocks)
- Metadata útil (tags do frontmatter, tipo de nota, status)
- Queries em PT-BR
- Casos limites (notas curtas, notas longas, MOCs vs concept notes)

Tem motivação real: o vault é usado todo dia, qualquer melhoria de qualidade é sentida imediatamente.

**Critério de sucesso:** consultar o próprio vault sem abrir o Obsidian, com qualidade decente; cada iteração de melhoria mensurável.

### Caminho 3 — RAG em stack profissional (quando aparecer oportunidade)

Se em projeto profissional (atual ou futuro) houver caso real de RAG, implementar usando o stack do próprio projeto. Mais demorado e menos didático que o Caminho 2, mas é o que se torna "RAG em produção" no CV. Se não há caso real ainda, não forçar — os Caminhos 1 e 2 já dão a base; o 3 espera a oportunidade certa.

**Critério de sucesso:** entrega no projeto com métricas, não estudo paralelo.

---

**Sugestão de ordem:** Caminho 1 → Caminho 2 → Caminho 3.

Pular o Caminho 1 e ir direto para o Caminho 3 é a forma mais comum de gastar 3 meses construindo RAG ruim porque os fundamentos ficaram opacos. Em qualquer caminho: **medir é não-negociável** (golden set + Ragas ou equivalente). Sem métrica, "parece funcionar" engana por meses.

## How to explain in English

### Short pitch

"RAG is how you make an LLM 'know' your data without training anything. You embed chunks of your documents, store them in a vector DB, retrieve relevant chunks per query, and inject them into the prompt. The trap is thinking RAG equals vector DB. Retrieval quality is where the work lives: chunking, hybrid search, reranking, query rewriting, and evaluation."

### Deeper version

"A reliable rule of thumb for 2026: nobody serious runs pure vector search in production. Hybrid search — BM25 plus vector — catches both semantic matches and exact terms. On top of that, a reranker on the top 20-30 results makes a bigger difference than most realize. And none of it matters if the chunks are garbage, so structure-aware chunking that respects headers, tables, and code blocks is the right place to start.

For evaluation, retrieval quality and generation quality should be measured separately. If retrieval fails, the LLM cannot save it. Ragas or similar frameworks give context precision, recall, and faithfulness — the three metrics that actually matter to monitor.

On the infra side, pgvector is a sensible default unless scale forces a dedicated vector DB. Keeping data in Postgres means filters, joins, and transactions work naturally. And metadata for filtering (tenant, date, doc type) is essential: multi-tenant RAG without metadata is a leak waiting to happen."

### Talking points

- "Pure vector search is a toy. Hybrid plus reranker is the default."
- "Chunking is where 50% of RAG quality lives."
- "Long context doesn't replace RAG — 'lost in the middle' is real."
- "RAG before fine-tuning, almost always."
- "Retrieval and generation should be measured separately."
- "Metadata is not optional — it's how you enforce permissions and freshness."

### Key vocabulary

- recuperação → retrieval
- aumentada → augmented
- geração → generation
- embutido / representação vetorial → embedding
- similaridade de cosseno → cosine similarity
- fragmentação → chunking
- sobreposição → overlap
- busca híbrida → hybrid search
- reordenação → reranking
- reescrita de query → query rewriting
- banco de dados vetorial → vector database
- índice → index
- filtro por metadados → metadata filter
- fidelidade → faithfulness
- precisão de contexto → context precision
- recall de contexto → context recall
- avaliação → evaluation
- busca por palavra-chave → keyword search

## Recursos

### Leitura fundamental

- [Pinecone — Learn RAG](https://www.pinecone.io/learn/retrieval-augmented-generation/)
- [Anthropic — Contextual retrieval](https://www.anthropic.com/news/contextual-retrieval) — técnica de chunking com contexto
- [RAG is Dead, Long Live RAG — Jerry Liu](https://www.llamaindex.ai/blog/rag-is-dead-long-live-rag) — análise do estado da arte
- [A Practitioner's Guide to RAG — LlamaIndex](https://www.llamaindex.ai/blog/a-practitioners-guide-to-retrieval-augmented-generation-rag)
- [Patterns for Building LLM-based Systems — Eugene Yan](https://eugeneyan.com/writing/llm-patterns/)
- [Lost in the Middle (paper)](https://arxiv.org/abs/2307.03172)
- [Contextual Retrieval (Anthropic blog)](https://www.anthropic.com/news/contextual-retrieval)

### Vector DBs docs

- [pgvector](https://github.com/pgvector/pgvector)
- [Pinecone Docs](https://docs.pinecone.io/)
- [Weaviate Docs](https://weaviate.io/developers/weaviate)
- [Qdrant Docs](https://qdrant.tech/documentation/)
- [Milvus Docs](https://milvus.io/docs)
- [LanceDB Docs](https://lancedb.github.io/lancedb/)

### Frameworks e tooling

- [LlamaIndex](https://www.llamaindex.ai/) — framework focado em RAG
- [LangChain RAG](https://python.langchain.com/docs/tutorials/rag/) — tutorial oficial
- [Ragas](https://docs.ragas.io/) — evaluation
- [TruLens](https://www.trulens.org/) — evaluation + observability
- [Cohere Rerank](https://docs.cohere.com/docs/rerank)
- [Voyage Embeddings + Rerank](https://docs.voyageai.com/)

### Benchmarks e papers

- [MTEB — Massive Text Embedding Benchmark](https://huggingface.co/spaces/mteb/leaderboard)
- [BEIR — benchmark de retrieval](https://github.com/beir-cellar/beir)
- [HyDE paper](https://arxiv.org/abs/2212.10496)
- [REALM paper](https://arxiv.org/abs/2002.08909)

## Deep dives — papers e técnicas avançadas

### RAG original (Lewis et al., 2020)

O paper que cunhou o termo "Retrieval-Augmented Generation". Combinou um retriever neural (DPR) com gerador (BART). A ideia central — buscar antes de gerar — virou o padrão. [arxiv](https://arxiv.org/abs/2005.11401)

### Dense Passage Retrieval (Karpukhin et al., 2020)

Paper que estabeleceu embeddings densos como superiores a BM25 para QA. Popularizou a arquitetura dual-encoder. [arxiv](https://arxiv.org/abs/2004.04906)

### ColBERT / SPLADE — late interaction e sparse learned retrieval

Alternativas a dense retrieval: ColBERT embedda cada token separadamente com late interaction; SPLADE aprende representações esparsas. Mais precisos que vector search em alguns benchmarks, mais caros. [ColBERT](https://arxiv.org/abs/2004.12832)

### HyDE — Hypothetical Document Embeddings (Gao et al., 2022)

Em vez de embed a query, gerar uma resposta hipotética com LLM e embed a resposta. Técnica de query rewriting simples e efetiva, muito usada em produção. [arxiv](https://arxiv.org/abs/2212.10496)

### Contextual Retrieval (Anthropic, 2024)

Pré-anexar contexto do documento em cada chunk antes de embedar. Resolve o problema de chunks isolados que perdem significado.

**Impacto reportado:** 49% redução em falhas de retrieval sozinho, 67% combinado com hybrid + rerank. [blog](https://www.anthropic.com/news/contextual-retrieval)

### GraphRAG (Microsoft, 2024)

Extrai entidades e relações com LLM, constrói grafo de conhecimento, combina retrieval com travessia. Destrava queries multi-hop. Caro de construir; use só quando justificado. [arxiv](https://arxiv.org/abs/2404.16130)

### Ragas — evaluation framework

Framework padrão para eval de RAG. Métricas: context precision, context recall, faithfulness, answer relevance, answer correctness. Separar eval de retrieval do eval de generation é crítico. [docs](https://docs.ragas.io/)

### Advanced patterns

- **Multi-query retrieval:** gerar N variações da query, unir resultados.
- **Parent document retrieval:** embed chunks pequenos, retornar chunks pais maiores.
- **Hierarchical retrieval:** busca primeiro em resumos, depois em chunks.
- **Corrective RAG (CRAG):** agent avalia se contexto é suficiente, busca mais se não for.
- **Self-RAG:** modelo decide quando retrievar e avalia contextos retornados.

## Casos comuns no mercado

Padrões frequentes em equipes implementando RAG em produção. Não são casos vividos pessoalmente — são armadilhas recorrentes documentadas em post-mortems, talks, e literatura técnica.

### Caso 1 — Chunking quebrado com tabelas

**Padrão observado:** base cresce, chunker fixed-size começa a quebrar tabelas e blocos estruturados no meio, criando chunks "lixo" misturados. Sintoma típico: respostas vagas em áreas específicas onde a documentação tem muito conteúdo tabular ou código.

**Fix típico:** chunker structure-aware respeitando headers/tabelas/code blocks; re-indexação completa após troca; Ragas regression test por categoria; chunker versionado como código.

**Lição:** chunking é onde metade da qualidade vive.

### Caso 2 — Embeddings misturados de versões diferentes

**Padrão observado:** upgrade de modelo de embeddings (ex: v2 → v3) aplicado apenas em novos docs. Docs antigos ficam com v2. Espaços vetoriais incompatíveis arruinam a qualidade do retrieval.

**Fix típico:** re-indexação completa ao trocar modelo; campo `embedding_version` nos metadados; pipeline detecta mismatch e bloqueia query híbrida entre versões.

**Lição:** não misturar embeddings de modelos diferentes na mesma base.

### Caso 3 — Multi-tenant leak

**Padrão observado:** feature multi-tenant implementada sem filtro estrito por `tenant_id` em todas as camadas. Bug em alguma query retorna chunks de outro tenant. Data leak confidencial — risco crítico em ambientes B2B/saúde/financeiro.

**Fix típico:** `tenant_id` obrigatório nos metadados; filtro no middleware da aplicação; Row-Level Security no Postgres (defense-in-depth); audit log cross-tenant que deve ser sempre zero.

**Lição:** isolamento multi-tenant é crítico e precisa ser multi-camada.

### Caso 4 — Reranker como otimização mais subestimada

**Padrão observado:** muitos sistemas em produção param em "hybrid search" e nunca adicionam reranker. Adicionar Cohere Rerank v3 ou bge-reranker-v2 em cima de top-20 tipicamente eleva context precision em 15-30 pontos percentuais, com custo de ~$0.002/query.

**Lição:** reranker é quase sempre worth it. Subestimado porque "mais um modelo no pipeline" parece complicação — o impacto em qualidade dilui isso.

### Caso 5 — Long context não substitui RAG

**Padrão observado:** com modelos de 1M+ tokens, equipes tentam "eliminar RAG" jogando tudo no contexto. Resultado típico: lost-in-the-middle severo (info no meio do contexto é mal usada), custo por query alto, latência inviável para muitos casos de uso.

**Lição:** context window grande não substitui retrieval bem feito. O retrieval continua valioso mesmo com 1M+ tokens, especialmente para precisão de citação e custo.

## Exercícios hands-on

### Lab 1 — RAG mínimo funcional

1. Stack: pgvector + OpenAI embeddings + Claude Sonnet.
2. Indexe ~50 docs próprios.
3. Chunking fixed como baseline.
4. Top-k=5, citation.
5. Golden set de 20 perguntas + Ragas.

### Lab 2 — Iteração medida de retrieval

Partindo do Lab 1, melhore de 0.6 para 0.85 context precision via:

1. Structure-aware chunking.
2. Hybrid search (BM25 + vector + RRF).
3. Reranker.
4. Query rewriting (HyDE).
5. Metadata filtering.

Gráfico mostrando ganho por iteração.

### Lab 3 — Contextual Retrieval

Implemente a técnica da Anthropic: LLM gera contexto curto por chunk, embedda `[contexto]+[chunk]`. Compare no golden set.

### Lab 4 — Eval pipeline profissional

Golden set + Ragas + GitHub Actions + dashboard de trend.

### Lab 5 — Multi-tenant isolado

Schema com `tenant_id` obrigatório, RLS no Postgres, teste de segurança cross-tenant.

### Lab 6 — Agentic RAG

Agent com tools `search`, `rerank`, `answer`, comparado com RAG fixo. Decida se o ganho justifica o custo.

## Veja também

- [[Inteligência Artificial]]
- [[LLMs]]
- [[Skills e Prompting]]
- [[Agents]]
- [[MCP]]
- [[Claude]]
- [[GitHub Copilot]]
- [[Codex]]
- [[Gemini]]
- [[Comparativo de LLMs]]
- [[Senda IA]]
