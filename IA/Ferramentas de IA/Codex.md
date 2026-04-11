---
title: "Codex"
created: 2026-04-01
updated: 2026-04-11
type: concept
status: evergreen
tags:
  - ia
  - llm
  - ferramentas
  - openai
  - codex
publish: true
---

# Codex

> Codex é a aposta da OpenAI em **coding agent cloud-based**: diferente de Claude Code ou Copilot (que rodam no seu IDE local), Codex opera em um sandbox na nuvem, recebe tarefas em linguagem natural, executa num ambiente isolado com seu repo clonado, e retorna um PR. Para um senior dev, o valor de Codex é diferente do uso diário: **tasks paralelas, trabalho assíncrono, e separação de concerns** — você descreve o que precisa, fecha o laptop, e volta horas depois com PRs prontos para review. Esta nota cobre o que Codex é em 2026 (muito diferente do "Codex 2021" original que era só o modelo), como usar, trade-offs, e quando faz sentido adotar. Para comparação com outras ferramentas, ver [[Comparativo de LLMs]]; para o padrão de agents em geral, [[Agents]].

## O que é

**Codex** em 2026 é o **coding agent cloud-native** da OpenAI. Essa é a encarnação atual do nome — o Codex original era um modelo (derivado de GPT-3) que alimentava a primeira versão do Copilot e foi descontinuado em 2023. Em 2024-2025 a OpenAI relançou "Codex" como **produto agent**, não como modelo.

### Capacidades principais

- **Receber tarefas em linguagem natural** via web UI, CLI, VS Code extension, ou API.
- **Executar em sandbox cloud isolado** — containers efêmeros com acesso ao repo.
- **Clonar e modificar repositórios GitHub** (com OAuth).
- **Rodar testes, builds, linters** no sandbox.
- **Abrir PRs automaticamente** com descrição gerada.
- **Processar múltiplas tasks em paralelo** — 5, 10, 20 tarefas simultâneas.
- **Codex Skills** — instruções reutilizáveis via `AGENTS.md`.
- **Integração com ChatGPT app** — pode ser invocado direto do app principal.
- **CLI local** (`codex`) — alternativa ao fluxo web quando você quer mais controle.

### O modelo por trás

Codex usa modelos OpenAI internos especializados em coding, derivados das famílias **GPT-4.x**, **o1** e **o3**. A OpenAI rotula de "Codex models" quando fine-tunados para tarefas agentic de código. Usuários não escolhem modelo diretamente na UI web; na API e CLI, pode-se selecionar entre variantes.

### As três modalidades de uso

1. **Codex Cloud (async)** — UI web ou ChatGPT, sandbox cloud, PR como output.
2. **Codex CLI (local)** — ferramenta no terminal, executa no seu ambiente local com sandbox leve.
3. **Codex em IDEs** — extension VS Code permite invocar tasks no cloud a partir do editor.

## O que diferencia um senior usando Codex

1. **Sabe quando usar Codex vs Claude Code vs Copilot.** Async/batch em Codex; interativo em Claude Code; inline em Copilot.
2. **Escreve `AGENTS.md` bem estruturado** — o equivalente ao CLAUDE.md para Codex.
3. **Escreve tasks com escopo claro** — Codex funciona melhor com descrições específicas do que com objetivos vagos.
4. **Delega em paralelo** tasks independentes para aproveitar a arquitetura async.
5. **Revisa cada PR gerado** — nunca merge sem human review, mesmo que todos os checks passem.
6. **Configura sandbox** com credentials apropriados (leituras de docs internos, acesso a CI, etc).
7. **Usa Codex Skills** para workflows recorrentes.
8. **Entende limites de sandbox** — não espera que Codex converse com serviços externos sem configuração.
9. **Mede custo por task** — Codex é cobrado por compute + tokens, pode ficar caro.
10. **Evita Codex para tarefas que exigem exploração interativa** — agent mode do Copilot ou Claude Code são melhores.

## Arquitetura e fluxo

```text
┌──────────────┐
│  Developer   │  1. Descreve task em natural language
│              │     "Add rate limiting to /api/v1/search"
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Codex UI /  │  2. Dispatcher cria sandbox container
│    API       │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Sandbox     │  3. Clona repo, instala deps
│  Container   │  4. Agent lê AGENTS.md e código
│              │  5. Planeja abordagem
│              │  6. Edita arquivos
│              │  7. Roda testes
│              │  8. Itera se falhar
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  GitHub PR   │  9. Abre PR com descrição + commits
│              │     + checks automáticos
└──────────────┘
```

Sandbox é efêmero — destruído após task completa. Cada task começa do zero, sem estado persistente (a menos que configurado via artifacts).

## AGENTS.md — a configuração crítica

O análogo do `CLAUDE.md` ou `copilot-instructions.md`. Arquivo markdown na raiz do repo que Codex lê como contexto permanente.

Em 2026, **AGENTS.md virou padrão de facto** adotado também por outras ferramentas (alguns frameworks o suportam como formato comum).

```markdown
# Agents instructions for this repo

This is a Go microservice using Echo framework and sqlx with PostgreSQL.

## Stack
- Go 1.23
- Echo web framework
- sqlx for DB access
- Testify for tests
- golangci-lint for linting

## Commands
- `make test` — run all tests
- `make lint` — run linter
- `make build` — build binary
- `make migrate` — run migrations

## Conventions
- Errors wrapped with fmt.Errorf("context: %w", err)
- No global state; use dependency injection
- All handlers return (res, error); errors mapped by middleware
- Table-driven tests for units
- One package per domain concept

## Architecture
- `cmd/` — entry points
- `internal/api/` — HTTP handlers
- `internal/service/` — business logic
- `internal/repo/` — data access
- `internal/domain/` — types and pure logic
- `pkg/` — public libraries (rare)

## Antipatterns
- panic() in library code
- Ignoring error returns
- SQL string concatenation (use parameterized queries)

## Testing conventions
- Integration tests in `*_integration_test.go` with build tag
- Mock external services with httptest
- Target >70% coverage on internal/service and internal/api
```

**Dica crítica:** AGENTS.md bem feita transforma drasticamente a qualidade do output. Um dev sênior investe 1-2h escrevendo bem no início do projeto.

## Codex Skills

Skills são instruções reutilizáveis que vivem em `.codex/skills/` ou diretamente referenciadas em `AGENTS.md`. Formato similar a outras ferramentas (frontmatter + conteúdo).

```markdown
---
name: add-endpoint
description: Add new REST endpoint with validation, tests, and docs
---

When asked to add a new endpoint:

1. Define input/output types in internal/domain/
2. Write validation in internal/api/validators/
3. Implement handler in internal/api/handlers/
4. Add route in internal/api/routes.go
5. Write unit tests for handler
6. Write integration test if touches DB
7. Update OpenAPI spec in docs/openapi.yaml
8. Add example curl in README
```

Skills aparecem como menu para o agent descobrir por demanda.

## Workflows reais com Codex

### Workflow 1 — Tasks paralelas em batch

Tenho 5 issues "pequenas" no backlog. Em vez de fazer sequencial:

1. Abro ChatGPT ou Codex UI.
2. Crio 5 tasks em paralelo, uma por issue, cada uma com link para a issue.
3. Fecho laptop, vou almoçar.
4. 1-2h depois, 5 PRs prontos para review.
5. Reviso sequencialmente, aprovando ou pedindo mudanças.

Isso destrava paralelismo de forma que nenhuma ferramenta local consegue — você não está esperando nem interagindo.

### Workflow 2 — Refactor amplo

Refactor que toca 50 arquivos (renomear um conceito, migrar lib):

1. Escrevo task detalhada explicando o "antes" e "depois".
2. Codex explora o repo, gera plano.
3. Revisa plano, ajusta.
4. Executa no sandbox, roda testes.
5. Abre PR com diff completo.

O ganho vs fazer no meu laptop: Codex persiste sem interrupção, sem distração, e é fácil lançar múltiplos refactors independentes em paralelo.

### Workflow 3 — Bug triage

Lista de 10 bugs relatados. Cada um vira uma task no Codex. Cada task investiga, tenta reproduzir, e ou: (a) abre PR com fix se claro, ou (b) retorna análise com info útil para human debugging.

### Workflow 4 — Dependency update

"Atualize todos os packages de prod para last stable minor, rode testes, abra PR com changelog" — vira uma task Codex recorrente que pode ser agendada.

## Codex CLI

Versão local do Codex. Roda no seu terminal, em ambiente do seu host (com sandboxing leve via container/firejail). Útil quando você quer:

- Controle fino sobre ambiente
- Network restrito
- Trabalhar offline parcialmente
- Debug local antes de escalar para cloud

```bash
# Instalação
npm install -g @openai/codex

# Uso básico
codex
> Add a --dry-run flag to the migration command
```

Similar em espírito ao Claude Code, mas focado em tarefas autoconfinadas vs conversação contínua.

## Diferenças com alternativas

### Codex vs Claude Code

| Aspecto | Codex | Claude Code |
| --- | --- | --- |
| **Execução** | Cloud sandbox (async) ou CLI local | Local CLI interativo |
| **Interação** | Fire-and-forget, volta ao PR | Conversacional contínuo |
| **Paralelismo** | Natural, N tasks em paralelo | Sequencial (multi-window possível) |
| **Contexto** | Repo clonado no sandbox | Filesystem local persistente |
| **Memory** | Efêmera por task | Persistente entre sessões |
| **Output** | PR/diff | Edições locais + commits |
| **Modelo** | OpenAI (GPT-4.1, o1, o3) | Anthropic (Opus/Sonnet) |
| **Configuração** | AGENTS.md + skills | CLAUDE.md + skills + hooks + MCP |
| **Integração MCP** | Crescente | Nativa e madura |
| **Custo** | Por task (compute + tokens) | Por tokens da API |

### Codex vs Copilot

- **Copilot** é integrado ao IDE; **Codex** é fire-and-forget cloud.
- **Copilot** brilha em inline e chat contextual; **Codex** brilha em tasks async completas.
- **Copilot workspace** é o mais próximo do Codex dentro da família GitHub.

### Quando escolher Codex

- **Tarefas paralelas que você quer delegar e checkar depois.**
- **Você vive no ChatGPT app** (Codex está integrado nativamente).
- **Equipes grandes** onde async scale importa mais que interatividade.
- **Repos grandes** onde sandbox com recursos dedicados ajuda.
- **Integrações CI-first** — Codex pode ser invocado de Actions para tasks automáticas.

### Quando não escolher

- **Exploração interativa** — use Claude Code ou Copilot Agent.
- **Features onde você precisa guiar decisões arquiteturais em tempo real.**
- **Projetos que precisam de contexto persistente complexo** — Codex começa do zero toda task.
- **Ambientes air-gapped ou com data residency estrita** — cloud pode não encaixar.
- **Budget sensível** — compute + tokens em tasks longas pode acumular.

## Segurança e limites

### Sandbox

Cada task roda em container isolado:

- Sem acesso a secrets externos (a menos que configurados)
- Network limitada por padrão (npm install sim, api calls externas não)
- Filesystem restrito ao clone do repo
- Destruído ao fim da task

### Credentials

- OAuth com GitHub para acesso ao repo
- Secrets do repo (GitHub Actions secrets) disponibilizados conforme política
- Data retention configurável (enterprise plans)

### Limites

- Tempo máximo por task (minutos a hora, depende do plano)
- Número de tasks simultâneas
- Compute quota

### Preocupações comuns

- **Data residency:** código vai para servidores OpenAI. Enterprise plans oferecem controles.
- **Proprietary code:** OpenAI compromete a não treinar com código de clientes enterprise.
- **Supply chain:** packages instalados no sandbox podem ser maliciosos; sandbox protege o ambiente mas não o código que você vai mergear.
- **Acesso a docs internos:** requer configuração (MCP, tools customizadas).

## Armadilhas comuns

### 1. Task vaga

"Melhore a API" → Codex devolve mudanças aleatórias. **Fix:** descrição específica, acceptance criteria claros.

### 2. Sem AGENTS.md

Agent trabalha com conhecimento genérico, não do projeto. **Fix:** AGENTS.md detalhado na raiz.

### 3. Merge sem review

Todos os checks passam → você mergea → bug sutil. **Fix:** human review sempre.

### 4. Paralelização de tasks dependentes

Task A e B tocam nos mesmos arquivos em paralelo → conflitos no merge. **Fix:** serializar dependencies.

### 5. Expectativa de contexto persistente

"Use o trabalho da task anterior" → Codex não tem. **Fix:** escrever task self-contained ou usar artifacts.

### 6. Ignorar custo

Task de 30 min com o1 → dezenas de dólares. **Fix:** budget por task, escolha de modelo justificada.

### 7. Usar Codex para exploração

"Entenda por que isso está lento" → Codex não vê ambiente, não consegue profile. **Fix:** usar Claude Code local.

### 8. Sem testes em projetos Codex-first

Codex gera código, testes ficam para trás. **Fix:** CI com coverage gate; Codex instruído a escrever teste primeiro.

### 9. Network restricto causa falha silenciosa

Task precisa acessar API interna não permitida. **Fix:** configurar proxy/allowlist ou rodar local.

### 10. Depender de um único agent

Codex como única ferramenta é engessamento. **Fix:** usar junto com Claude Code e Copilot em papéis diferentes.

## Na prática (da minha experiência)

No MedEspecialista, Codex entrou como ferramenta complementar em 2025 para workflows específicos:

**Uso 1 — batch refactors.** Tínhamos um refactor de renomear um conceito de domínio que afetava ~60 arquivos. Escrevi task detalhada com before/after, deleguei para Codex, fui almoçar. Voltei com PR pronto, revisei, mergeei. Ganho: ~3h de trabalho manual comprimidas em ~1h de wait + review.

**Uso 2 — dependency updates recorrentes.** Agendei Codex task semanal para tentar update de minor versions + rodar testes. PRs aparecem toda segunda, reviso. Reduziu "dependency debt" para quase zero sem esforço humano.

**Uso 3 — bug triage em batch.** Quando backlog de "bugs pequenos" cresce, delego 5-10 em paralelo. Retornos variam: ~30% viram PR direto, ~40% viram análise que acelera o fix humano, ~30% não dão em nada (task vaga ou bug complexo).

**Uso 4 — CI para tarefas automatizáveis.** Action no GitHub dispara Codex para: escrever release notes do PR, gerar resumo de mudanças na docs, triar issues com labels.

**Lições:**

- **AGENTS.md bem feito é a diferença entre "funciona" e "não funciona".**
- **Async é uma ferramenta mental, não só técnica.** Você aprende a pensar "o que posso delegar e checar depois".
- **Paralelização real destrava throughput.** 5 tasks em paralelo rendem mais que 5 sequenciais porque não tem task-switching.
- **Custo escala com uso.** Fácil gastar $50-100/mês em workflows pequenos; enterprise pode chegar a milhares.
- **Human review sempre.** Codex erra de formas novas e criativas.

**Incidente memorável:** uma task Codex de "add input validation" aplicou validação agressiva demais e quebrou clientes antigos que mandavam campos opcionais como `null`. Passou testes (os testes não cobriam esse caso) mas quebrou em produção. Dentro de 30 min após deploy. Fix: rollback + teste de regressão + lição aprendida de sempre revisar _semantics_ do PR, não só os checks.

**Outro incidente:** deleguei 8 tasks em paralelo, 3 delas tocavam arquivos sobrepostos. 2 PRs limparam no merge, 1 falhou com conflict. Depois passei a serializar tasks com overlap de path.

## How to explain in English

### Short pitch

"Codex is OpenAI's cloud-native coding agent. Unlike Claude Code or Copilot, it runs in a sandbox in the cloud, clones your repo, and returns a PR. The value for me is async and parallel work — I can fire off five tasks, close the laptop, and come back to five PRs ready for review. It complements my interactive tools rather than replacing them."

### Deeper version

"Codex has three modes: cloud async via the web UI or ChatGPT app, local CLI for more controlled environments, and IDE integrations. Each task runs in an isolated container — clone, install deps, read AGENTS.md, plan, execute, open a PR. No persistent state between tasks; each one starts fresh.

Where Codex wins is async scale. I have five small issues in the backlog? I dispatch five tasks in parallel, and review PRs later. I need a 60-file refactor? I write a detailed task and let Codex grind while I do other work. It's a different posture from interactive coding — fire, forget, review.

The configuration that matters is AGENTS.md, which is becoming something of a standard across tools — similar in role to CLAUDE.md or copilot-instructions.md. Good AGENTS.md means Codex understands architecture, conventions, and commands without me re-explaining every time.

The limits to be clear on: no persistent context between tasks, sandbox has restricted network, and you pay for compute plus tokens — so long complex tasks can add up. And you always human-review the PR, because agents fail in creative ways that CI doesn't catch."

### Talking points

- "Async and parallel is the killer feature. Not interactive speed."
- "AGENTS.md is standard now — invest in it."
- "Always human-review, even when all checks pass."
- "Codex complements Claude Code and Copilot, it doesn't replace them."
- "Sandbox limits are features — they keep your host clean."

### Key vocabulary

- agente de código → coding agent
- caixa de areia → sandbox
- execução assíncrona → asynchronous execution
- tarefa em lote → batch task
- paralelização → parallelization
- efêmero → ephemeral
- clonagem de repositório → repository cloning
- contêiner isolado → isolated container

## Recursos

### Oficial

- [OpenAI Codex](https://openai.com/codex/) — página oficial
- [Codex Docs](https://developers.openai.com/codex/)
- [Codex Skills](https://developers.openai.com/codex/skills/)
- [Codex CLI](https://github.com/openai/codex) — projeto open source
- [Agents.md guide](https://www.aihero.dev/a-complete-guide-to-agents-md)

### Blogs e análises

- [Pragmatic Engineer — AI tools](https://newsletter.pragmaticengineer.com/)
- [Simon Willison — OpenAI posts](https://simonwillison.net/tags/openai/)

## Veja também

- [[LLMs]]
- [[Agents]]
- [[Skills e Prompting]]
- [[Claude]]
- [[GitHub Copilot]]
- [[Gemini]]
- [[Comparativo de LLMs]]
- [[Inteligência Artificial]]
- [[MCP]]
- [[Trilha IA]]
