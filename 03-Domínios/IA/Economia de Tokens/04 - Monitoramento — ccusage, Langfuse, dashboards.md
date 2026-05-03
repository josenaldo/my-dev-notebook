---
title: "Monitoramento — ccusage, Langfuse, dashboards"
created: 2026-05-02
updated: 2026-05-02
type: concept
status: seedling
publish: true
tags:
  - economia-tokens
  - ia
  - custos
aliases:
  - ccusage
  - Langfuse
  - LLM monitoring
  - Token monitoring
---

# Monitoramento — ccusage, Langfuse, dashboards

> [!abstract] TL;DR
> Monitorar tokens é o passo zero da economia — sem dados, otimização é adivinhação. Em 2026, três camadas de monitoramento cobrem todos os cenários: **ccusage** (CLI local para Claude Code), **Helicone** (proxy fácil de instalar), e **Langfuse** (observability open-source com tracing profundo). O mínimo viável é logar `usage.input_tokens` + `usage.output_tokens` de cada chamada e somar por dia/projeto.

## O que é

Monitoramento de tokens é o processo de coletar, agregar e visualizar dados de consumo de tokens para identificar desperdícios e medir o impacto de otimizações.

## Como funciona

### Camada 1: Dashboard do provider (nativo)

| Provider  | Onde                      | O que mostra                            |
| --------- | ------------------------- | --------------------------------------- |
| Anthropic | console.anthropic.com     | Gasto diário, por modelo, por workspace |
| OpenAI    | platform.openai.com/usage | Gasto por dia, por modelo, API keys     |
| Google    | Cloud Console / Vertex AI | Gasto por projeto, por modelo           |

**Limitação:** visão agregada, sem detalhes por sessão ou por tarefa.

### Camada 2: ccusage (para Claude Code)

```bash
# Instalar
npm install -g ccusage

# Analisar uso do Claude Code
ccusage

# Output:
# Date       | Input    | Output   | Cache Read | Cost
# 2026-05-01 | 450,000  | 85,000   | 320,000    | $3.42
# 2026-05-02 | 680,000  | 120,000  | 510,000    | $4.85
# Total      | 1,130,000| 205,000  | 830,000    | $8.27

# Filtrar por projeto
ccusage --project ~/repos/estudeme

# Ver detalhes por sessão
ccusage --sessions
```

ccusage lê os logs JSONL locais do Claude Code — funciona offline, sem proxy.

### Camada 3: Helicone (proxy — setup rápido)

```python
# Mudar apenas a base URL
import anthropic

client = anthropic.Anthropic(
    base_url="https://anthropic.helicone.ai/v1",
    default_headers={
        "Helicone-Auth": f"Bearer {HELICONE_API_KEY}"
    }
)

# Tudo funciona igual — Helicone loga automaticamente
response = client.messages.create(...)
```

**Vantagem:** setup em 5 minutos, sem mudança de código além da base URL.
**Dashboard:** visão por request, custo acumulado, latência, cache hit rate.

### Camada 4: Langfuse (observability profunda)

```python
from langfuse import Langfuse
from langfuse.decorators import observe

langfuse = Langfuse()

@observe(name="fix-bug")
def fix_bug(issue_description):
    # Langfuse rastreia automaticamente:
    # - Tokens de input/output
    # - Latência
    # - Custo calculado
    # - Trace completo da cadeia de chamadas
    response = client.messages.create(...)
    return response
```

**Vantagens:**

- Open source (MIT), self-hostable
- Tracing de cadeias de chamadas (agente → tool → sub-call)
- Custo calculado automaticamente por modelo
- Integração com avaliação de qualidade

### Comparativo de ferramentas

| Ferramenta             | Setup  | Custo              | Granularidade     | Melhor para                |
| ---------------------- | ------ | ------------------ | ----------------- | -------------------------- |
| **Provider dashboard** | Zero   | Grátis             | Diário/modelo     | Visão geral de billing     |
| **ccusage**            | 1 min  | Grátis             | Sessão/projeto    | Usuários de Claude Code    |
| **Helicone**           | 5 min  | Freemium           | Request/feature   | Setup rápido, caching      |
| **Langfuse**           | 30 min | Grátis (self-host) | Trace/span        | Observability profunda     |
| **Braintrust**         | 1h     | Pago               | Trace + qualidade | Correlação custo-qualidade |
| **Datadog**            | 2h+    | Pago               | Enterprise-grade  | Já usa Datadog             |

### Métricas a monitorar

| Métrica                | O que indica                      | Meta                        |
| ---------------------- | --------------------------------- | --------------------------- |
| **Custo por sessão**   | Eficiência geral                  | <$5 por feature             |
| **Custo por turn**     | Se o contexto está explodindo     | Estável (não crescente)     |
| **Cache hit rate**     | Eficácia do prompt caching        | >60%                        |
| **Input/output ratio** | Se o input está inflado           | <10:1                       |
| **Retries por sessão** | Quantas vezes o agente erra       | <20%                        |
| **Reasoning tokens**   | Se thinking budget está calibrado | Proporcional à complexidade |

### Setup mínimo viável

Se não quer instalar nada, apenas adicione logging:

```python
import json, datetime

def log_usage(response, task_name):
    usage = response.usage
    log = {
        "timestamp": datetime.datetime.now().isoformat(),
        "task": task_name,
        "model": response.model,
        "input_tokens": usage.input_tokens,
        "output_tokens": usage.output_tokens,
        "cache_read": getattr(usage, 'cache_read_input_tokens', 0),
        "cost": (usage.input_tokens * 3 + usage.output_tokens * 15) / 1_000_000
    }
    with open("llm_usage.jsonl", "a") as f:
        f.write(json.dumps(log) + "\n")
```

## Armadilhas

- **Não monitorar** — o erro mais caro. Sem dados, otimização é impossível.
- **Monitorar só o total** — saber que gastou $50/dia é inútil sem saber ONDE gastou.
- **Ignorar cache hit rate** — se configurou caching mas o hit rate é 10%, algo está errado.
- **Setup over-engineered** — para dev solo, ccusage ou uma planilha basta. Langfuse é para times.

## Veja também

- [[02 - Anatomia do gasto — input, output e reasoning]] — o que monitorar
- [[05 - Prompt caching na prática]] — a primeira otimização a medir
- [[15 - Orçamento e hard limits]] — como definir metas

## Referências

- **ccusage** — *GitHub Repository* (2026). Ferramenta CLI para Claude Code.
- **Langfuse** — *Documentation* (langfuse.com). Platform open-source.
- **Helicone** — *Documentation* (helicone.ai). Proxy de observability.
