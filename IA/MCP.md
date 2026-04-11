---
title: "MCP (Model Context Protocol)"
created: 2026-04-11
updated: 2026-04-11
type: concept
status: evergreen
tags:
  - ia
  - mcp
  - agents
  - entrevista
publish: true
---

# MCP (Model Context Protocol)

> MCP é, em uma frase, o **"USB-C para agents de IA"**. Antes dele, cada integração entre LLM e um sistema externo (banco de dados, filesystem, Jira, Slack, seu próprio serviço) era reinventar a roda — cada cliente (Claude, Cursor, Copilot) tinha seu próprio formato de plugin, cada servidor expunha de um jeito. MCP, lançado pela Anthropic em novembro de 2024 e rapidamente adotado por OpenAI, Google, Microsoft e a comunidade, padroniza essa conexão. Em 2026, se você está construindo aplicação com agents, MCP é infraestrutura básica, como HTTP.

## O que é

**Model Context Protocol** é um protocolo aberto que define como **clientes de IA** (Claude Desktop, Claude Code, Cursor, VS Code, Copilot, IDEs) se comunicam com **servidores de contexto** (filesystem, git, database, APIs, seu sistema proprietário). É um padrão JSON-RPC sobre transporte stdio ou HTTP+SSE.

A analogia oficial: **"MCP é para agents o que o Language Server Protocol foi para editores"**. Antes do LSP, cada editor tinha integração própria com cada linguagem. LSP padronizou e destravou o ecossistema de editores e tools. MCP faz o mesmo para IA.

### Os três papéis

```text
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│                 │       │                 │       │                 │
│   MCP Client    │◄─────►│   MCP Protocol  │◄─────►│   MCP Server    │
│                 │       │   (JSON-RPC)    │       │                 │
│ Claude Desktop  │       │                 │       │   filesystem    │
│ Claude Code     │       │  stdio or       │       │   git           │
│ Cursor          │       │  HTTP+SSE       │       │   postgres      │
│ VS Code Copilot │       │                 │       │   slack         │
│                 │       │                 │       │   custom        │
└─────────────────┘       └─────────────────┘       └─────────────────┘
```

- **Client:** aplicação que hospeda o LLM e descobre capacidades via MCP.
- **Server:** expõe capabilities (tools, resources, prompts) ao client.
- **Host:** a aplicação que roda ambos (Claude Desktop é client + host).

## O que diferencia um senior em MCP

1. **Entende MCP como protocolo, não como feature específica de Claude.** É aberto e suportado por múltiplos clientes.
2. **Sabe quando escrever um MCP server vs uma tool direta.** MCP para capabilities reusáveis entre múltiplos clients; tool direta quando só um client usa.
3. **Distingue tools, resources e prompts** — os três primitivos do MCP — e sabe usar cada um.
4. **Desenha permissões e isolação.** MCP server rodando com acesso total a filesystem é risco de segurança.
5. **Implementa MCP stdio vs HTTP com consciência.** Local vs remoto têm trade-offs diferentes.
6. **Versiona capabilities** e lida com evolução de schema.
7. **Observa tool calls via MCP,** instrumentando logs centralizados.
8. **Conhece o ecossistema:** servers oficiais, community servers, registries.
9. **Escreve servers em linguagem adequada** — Python/TypeScript SDK oficiais, mas Go/Rust também viável.
10. **Trata MCP como superfície de ataque** — prompt injection, capability abuse, supply chain.

## Por que MCP existe

Antes de MCP, se você queria dar ao Claude acesso ao seu filesystem, ao Git, a um Postgres e a um CRM:

- Claude Desktop tinha seu formato de plugin.
- Cursor tinha outro.
- Copilot extensions, outro.
- ChatGPT plugins (já descontinuados), outro.
- Cada tool precisava ser escrita N vezes.

Com MCP:

- Escreve-se **um servidor MCP** que expõe capability.
- Qualquer client compatível pode usar.
- Comunidade cria servers, reutiliza-se.
- Novos clients entram no ecossistema sem re-implementar nada.

**O momento-chave** foi quando a OpenAI aderiu em 2025 e o ChatGPT app support a MCP. Virou padrão de facto.

## Os três primitivos do MCP

MCP servers expõem três tipos de capability:

### 1. Tools — o que o modelo pode **fazer**

Funções que o LLM pode chamar via tool use. Cada tool tem nome, descrição, input schema.

```json
{
  "name": "query_database",
  "description": "Run a read-only SQL query against the analytics database",
  "inputSchema": {
    "type": "object",
    "properties": {
      "sql": { "type": "string", "description": "SELECT query only" }
    },
    "required": ["sql"]
  }
}
```

Tools são **ativas** — o modelo decide chamar, servidor executa, retorna resultado.

### 2. Resources — o que o modelo pode **ler**

Conteúdo estático ou semi-estático que o client pode expor ao LLM como contexto. Identificados por URI. Podem ser texto, binário, JSON.

```json
{
  "uri": "file:///project/README.md",
  "name": "Project README",
  "mimeType": "text/markdown"
}
```

O client decide quando incluir resource no contexto. Resources são **passivos** — não executam, só existem para leitura.

Exemplos típicos:

- Arquivos do projeto
- Schemas de banco
- Documentação
- Issues do GitHub
- Anotações

### 3. Prompts — workflows reutilizáveis

Templates de prompt parametrizados que o client pode invocar. Diferente de tools, prompts são **para o usuário escolher**, não para o modelo decidir.

```json
{
  "name": "code-review",
  "description": "Review a file for bugs and style",
  "arguments": [
    { "name": "file", "description": "File to review", "required": true }
  ]
}
```

O usuário escolhe "code-review" num menu, passa o arquivo, e o servidor retorna o prompt formatado para o LLM.

## Arquitetura — como funciona na prática

### Fluxo de conexão

```text
1. Client inicia server (stdio) ou conecta (HTTP+SSE)
2. Handshake: client envia InitializeRequest com client info + capabilities
3. Server responde com server info + capabilities que oferece
4. Client pede listagem: tools/list, resources/list, prompts/list
5. Client usa capabilities conforme necessário
```

### Transportes

**stdio** (standard input/output):

- Server é processo local (subprocess do client).
- JSON-RPC sobre stdin/stdout.
- Usado para servers locais (filesystem, git local, scripts).
- Simples, seguro, isolado por processo.

**HTTP + Server-Sent Events (SSE):**

- Server é serviço HTTP.
- Client faz requests HTTP para enviar, recebe stream SSE para notificações.
- Usado para servers remotos (APIs, DBs, serviços cloud).
- Requer auth, TLS, etc.

**WebSocket:** algumas implementações, menos padrão.

### JSON-RPC em 10 segundos

Protocolo RPC bidirecional baseado em JSON. Cada mensagem é request, response ou notification.

```json
// Request
{"jsonrpc": "2.0", "id": 1, "method": "tools/list"}

// Response
{"jsonrpc": "2.0", "id": 1, "result": {"tools": [...]}}
```

## Exemplos de MCP servers oficiais

Anthropic e comunidade mantêm servers prontos. Alguns dos mais usados:

| Server | O que expõe | Uso típico |
| --- | --- | --- |
| **filesystem** | read/write/list arquivos locais | Claude edita código |
| **git** | git operations (log, diff, commit) | Review e gerenciamento |
| **github** | issues, PRs, actions via GitHub API | Workflow devops |
| **postgres** | SQL queries em DBs | Análise de dados |
| **sqlite** | SQL em sqlite files | Analytics rápido |
| **slack** | Enviar msgs, ler canais | Notificações |
| **google-drive** | Listar, ler docs do GDrive | Consultar arquivos corporativos |
| **brave-search** | Web search via Brave API | Research |
| **puppeteer** | Controle de browser | Scraping, automation |
| **memory** | Armazena fatos/notas persistentes | Memória do agent |
| **sequentialthinking** | Structured reasoning | Pensamento passo a passo |
| **fetch** | HTTP requests genéricos | APIs ad-hoc |
| **everything-search** | Busca full-text no filesystem | Code exploration |

Lista oficial: [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers).

Para instalar no Claude Desktop, edita-se um `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://..."
      }
    }
  }
}
```

## Construindo um MCP server

### SDKs oficiais

- **TypeScript/JavaScript:** `@modelcontextprotocol/sdk`
- **Python:** `mcp` (pypi)
- Comunidade: Go, Rust, Java, C#, etc.

### Exemplo mínimo — TypeScript

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "my-weather-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "get_weather",
      description: "Get current weather for a city",
      inputSchema: {
        type: "object",
        properties: {
          city: { type: "string", description: "City name" },
        },
        required: ["city"],
      },
    },
  ],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "get_weather") {
    const city = request.params.arguments?.city as string;
    const weather = await fetchWeather(city);
    return {
      content: [{ type: "text", text: JSON.stringify(weather) }],
    };
  }
  throw new Error("Unknown tool");
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Exemplo mínimo — Python

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

server = Server("my-weather-server")

@server.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="get_weather",
            description="Get current weather for a city",
            inputSchema={
                "type": "object",
                "properties": {
                    "city": {"type": "string"}
                },
                "required": ["city"]
            }
        )
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "get_weather":
        weather = await fetch_weather(arguments["city"])
        return [TextContent(type="text", text=str(weather))]
    raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read, write):
        await server.run(read, write, server.create_initialization_options())

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

### Adicionando resources e prompts

```python
@server.list_resources()
async def list_resources() -> list[Resource]:
    return [Resource(
        uri="weather://cities",
        name="Supported cities",
        mimeType="application/json"
    )]

@server.read_resource()
async def read_resource(uri: str) -> str:
    if uri == "weather://cities":
        return json.dumps(SUPPORTED_CITIES)
    raise ValueError("Unknown resource")

@server.list_prompts()
async def list_prompts() -> list[Prompt]:
    return [Prompt(
        name="weather_report",
        description="Generate weather report for multiple cities",
        arguments=[
            PromptArgument(name="cities", required=True)
        ]
    )]

@server.get_prompt()
async def get_prompt(name: str, arguments: dict) -> GetPromptResult:
    if name == "weather_report":
        cities = arguments["cities"]
        return GetPromptResult(
            messages=[
                PromptMessage(
                    role="user",
                    content=TextContent(
                        type="text",
                        text=f"Generate a weather report for: {cities}"
                    )
                )
            ]
        )
```

## MCP remoto — HTTP + SSE

Servers locais (stdio) são fáceis e seguros. Servers remotos são mais complexos mas destravam capabilities que precisam rodar em ambiente próprio (DBs corporativos, APIs privadas).

### Arquitetura

```text
Client          HTTP Server
  │                 │
  │──POST /msg─────►│  (request/notification)
  │                 │
  │◄────SSE stream──│  (responses e notifications)
```

### Auth

Suporta múltiplos mecanismos:

- **OAuth 2.0** (recomendado para servers públicos)
- **Bearer tokens**
- **mTLS**
- **API keys**

Especificação detalha negociação durante Initialize.

### Quando usar HTTP

- Server precisa rodar em infra própria (DB cluster, cluster corporativo).
- Multi-user: vários clients conectam ao mesmo server.
- Capability cara de instanciar (mantém estado entre requests).

### Quando usar stdio

- Single-user, local.
- Server precisa de acesso ao filesystem/ambiente do usuário.
- Latência mínima.
- Simples de distribuir (um binário/script).

**Em 2026, maioria dos servers community é stdio.** HTTP emergiu em servers corporativos.

## Segurança — o ponto que ninguém quer pensar

MCP servers têm acesso real ao sistema. **Filesystem server pode deletar tudo.** **Git server pode force-push.** **Shell server executa código arbitrário.** Com um LLM no controle, sob influência potencial de prompt injection, o risco é significativo.

### Modelo de ameaças

- **Prompt injection:** conteúdo externo faz o LLM chamar tools destrutivas.
- **Capability escalation:** tool "inocente" usada de forma criativa para dano.
- **Data exfiltration:** agent lê dados sensíveis e manda para fora.
- **Supply chain:** MCP server malicioso instalado via npm/pypi.

### Defesas

1. **Princípio do menor privilégio.** Filesystem server com escopo restrito (`/project/src`, não `/`). Postgres server com usuário read-only.
2. **Allowlist de tools.** Client deve permitir habilitar tools individualmente, não "tudo ou nada".
3. **Human-in-the-loop para tools destrutivas.** Claude Desktop pede confirmação em operações sensíveis (configurável).
4. **Audit log** de todas chamadas MCP.
5. **Sandboxing** quando possível: Docker, Firecracker, nsjail.
6. **Review de servers community.** Não rode servers aleatórios sem olhar código.
7. **Auth em servers HTTP.** TLS obrigatório, tokens rotacionáveis.
8. **Rate limiting** para evitar abuse.

### Boas práticas de desenho de server

- **Não exponha tools destrutivas se não forem necessárias.**
- **Tools destrutivas explícitas no nome** (ex: `delete_file`, não `cleanup`).
- **Retornar confirmação para ações destrutivas**, não executar silenciosamente.
- **Limite escopo de acesso** via parâmetros de inicialização.
- **Documentar riscos** no README.

## Ecossistema em 2026

### Clients compatíveis

- **Claude Desktop** (Anthropic) — o primeiro client.
- **Claude Code** (CLI) — suporte nativo.
- **Cursor** — suporte via config.
- **VS Code Copilot** — suporte nas versões recentes.
- **Windsurf** — suporte nativo.
- **Zed** — suporte crescente.
- **Gemini CLI / Gemini Code Assist** — suporte crescente.
- **OpenAI ChatGPT app** — suporte após adoção em 2025.

### Registries e marketplaces

- [Official Servers Repo](https://github.com/modelcontextprotocol/servers)
- [MCP Server Registry](https://mcp.so/)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- [Smithery](https://smithery.ai/) — marketplace com installer

### Spec

- [modelcontextprotocol.io](https://modelcontextprotocol.io/) — spec oficial
- [GitHub modelcontextprotocol](https://github.com/modelcontextprotocol) — SDKs, servers, spec

## Armadilhas comuns

### 1. Usar MCP onde uma tool direta bastaria

Se o server só será usado por uma aplicação, escrever tool direto pode ser mais simples. MCP brilha em **reusabilidade**.

### 2. Expor servers sem auth

HTTP MCP server sem auth é um backdoor. TLS + tokens sempre.

### 3. Não versionar capabilities

Mudou schema de uma tool? Clients antigos quebram. **Fix:** versioning explícito, tools novas com nomes novos.

### 4. Outputs de tool gigantes

Tool retorna 50K tokens de JSON. Client inunda contexto do LLM. **Fix:** paginar, compactar, retornar resumos.

### 5. Ignorar cancelation

Tools longas sem suporte a cancelamento são chatas. **Fix:** respeitar cancellation request do protocolo.

### 6. Logs locais só

Sem observabilidade central, debug é dor. **Fix:** instrumentar com OpenTelemetry ou similar.

### 7. Tratar servers community como código confiável

Review antes de rodar. Package do npm/pypi pode ser malicioso.

### 8. Esquecer que MCP é JSON-RPC

Erros precisam seguir padrões JSON-RPC. Retornar HTTP 500 em vez de JSON-RPC error é bug.

## Na prática (da minha experiência)

No MedEspecialista, MCP entrou no workflow em 2025, quando o Claude Code passou a suportar.

**Uso 1 — Desenvolvimento local.** Rodo Claude Code com os servers: filesystem (escopo no projeto), git, postgres (DB de dev, read-only), github (issues, PRs). O ganho é que esses capabilities ficam disponíveis em qualquer projeto sem configuração duplicada.

**Uso 2 — Server customizado para o domínio.** Escrevi um MCP server que expõe tools específicas do MedEspecialista: `list_patients`, `read_consultation`, `search_procedures`. Isso vira utilidade para coding assistants quando precisam testar código contra dados reais (com sanitização).

**Uso 3 — Server interno de observabilidade.** Um MCP server que conecta no Langfuse/Grafana para o agent consultar métricas durante debugging. "Qual o p99 de latência do endpoint X?" vira uma tool call.

**Lições:**

- **MCP server bem-feito é reutilizado em múltiplos workflows.** Investi 1 dia no filesystem server do MedEspecialista e uso em todos projetos.
- **Allowlist de tools no client é crítico.** Em projetos sensíveis desabilito tools destrutivas na config.
- **HTTP + OAuth é complexidade significativa.** Só vale se o server precisa ser compartilhado.
- **Debugging de MCP é sofrível sem logs.** Instrumentar desde o início.

**Incidente memorável:** um server community que instalei (um "time tracking" server) fazia log dos prompts no próprio servidor deles. Descobri ao reparar requests HTTP saindo durante uso normal. Desinstalei imediatamente. **Lição:** review code antes de rodar server community, mesmo pequenos. Confie mas verifique, em escala pequena.

## How to explain in English

### Short pitch

"MCP is the standard protocol for connecting LLM clients to external capabilities — filesystem, databases, APIs, custom services. Think 'LSP for AI'. You write a server once, any MCP-compatible client can use it. Claude Code, Cursor, Copilot, and ChatGPT all support it in 2026."

### Deeper version

"The value of MCP is composability. Before it, every tool had to be written as a plugin for each client, in each client's format. MCP gives you one server, any client. I've written a few custom servers for the codebase at my company — one for our internal knowledge base, one for our observability stack — and they just work whether I'm in Claude Code, Cursor, or testing something in Claude Desktop.

The protocol itself is JSON-RPC 2.0 over either stdio or HTTP+SSE. stdio for local servers, HTTP for remote. The primitives are tools, resources, and prompts — tools being the active ones the LLM decides to invoke.

The thing I think a lot of people underestimate is the security model. MCP servers often have real capabilities — filesystem access, database queries, API calls. Combined with prompt injection risks from any external content the LLM consumes, you need least-privilege, allowlists, human-in-the-loop for destructive actions, and supply chain vigilance. I audit every third-party server I install."

### Phrases to use

- "MCP is LSP for AI tools."
- "One server, many clients — that's the promise."
- "Three primitives: tools, resources, prompts."
- "Security is not an afterthought — least privilege, allowlists, human-in-the-loop for destructive actions."
- "stdio for local, HTTP+SSE for remote."

### Key vocabulary

- protocolo → protocol
- servidor → server
- cliente → client
- hospedeiro → host
- ferramenta → tool
- recurso → resource
- chamada remota de procedimento → remote procedure call (RPC)
- transporte → transport
- capacidade → capability
- negociação → handshake / negotiation
- escopo → scope
- princípio do menor privilégio → principle of least privilege
- cadeia de suprimentos → supply chain

## Recursos

### Spec e docs oficiais

- [modelcontextprotocol.io](https://modelcontextprotocol.io/)
- [GitHub: modelcontextprotocol](https://github.com/modelcontextprotocol)
- [Spec repository](https://github.com/modelcontextprotocol/specification)
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk)

### Servers

- [Official Servers](https://github.com/modelcontextprotocol/servers)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- [MCP.so Registry](https://mcp.so/)
- [Smithery marketplace](https://smithery.ai/)

### Tutoriais e blog posts

- [Introducing MCP — Anthropic](https://www.anthropic.com/news/model-context-protocol)
- [Building a MCP Server — tutorial](https://modelcontextprotocol.io/tutorials/building-mcp-server)
- [MCP Explained — Simon Willison](https://simonwillison.net/tags/mcp/)

## Veja também

- [[Inteligência Artificial]]
- [[LLMs]]
- [[Agents]] — MCP é infraestrutura para agents
- [[Skills e Prompting]]
- [[RAG e Vector Databases]]
- [[Claude]] — primeiro client de MCP
- [[GitHub Copilot]]
- [[Codex]]
- [[Gemini]]
- [[Comparativo de LLMs]]
- [[Trilha IA]]
