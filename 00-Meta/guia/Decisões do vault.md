---
title: "Decisões do vault"
created: 2026-04-28
updated: 2026-04-28
type: reference
status: seedling
tags:
  - guia
  - meta
  - adr
publish: false
---

# Decisões do vault

Registro de decisões de design do Codex — o **porquê** de escolhas que ficaram. ADRs leves: cada decisão tem contexto, escolha e consequências.

Use este arquivo quando: estiver tentado a mudar uma convenção e quiser saber por que ela existe; quiser registrar uma nova decisão pra que futuro-você ou colaboradores entendam.

## Formato

Cada decisão segue este template:

```markdown
## [Título da decisão]

- **Data:** YYYY-MM-DD
- **Status:** ativa | revisada | abandonada

**Contexto:** o que motivou a decisão (problema, restrição, hipótese).

**Escolha:** o que foi decidido.

**Consequências:** o que isso implica (positivo e negativo).
```

---

## Decisões registradas

### Separar conteúdo público e apocrypha em repositórios distintos

- **Data:** 2026-04 (anterior à criação deste registro — preencher data exata se relevante)
- **Status:** ativa

**Contexto:** *(preencher: o que motivou a separação. Compliance, dados sensíveis de clientes, exposição em entrevistas, etc.)*

**Escolha:** apocrypha vive em `codex-technomanticus-apocrypha` (privado), público vive em `codex-technomanticus`. O site Quartz só faz checkout do público — apocrypha sequer chega na máquina de build.

**Consequências:**

- Eliminação de risco de vazamento por config errada de `publish` (a separação é física, não lógica).
- Custo: dois vaults pra manter, atenção redobrada com wikilinks que cruzam repos.
- Regra derivada: nenhum wikilink ou menção a apocrypha em arquivo público. Detalhes em [[Publicação]].

---

### *(decisões abaixo são candidatas — preencher quando o porquê for registrado)*

### As 5 zonas (00-Meta, Pergaminhos, Glosas, Domínios, Sendas)

- **Data:** *preencher*
- **Status:** ativa

**Contexto:** *preencher — por que 5 zonas e não 3 ou 7? Por que o pipeline cognitivo bruto→destilado→integrado→curatorial?*

**Escolha:** estrutura em 5 zonas numeradas conforme [[Como usar este vault]].

**Consequências:** *preencher.*

---

### Nomenclatura mística (Pergaminhos, Glosas, Domínios, Sendas, grimório)

- **Data:** *preencher*
- **Status:** ativa

**Contexto:** *preencher — vault como grimório, eng. de software como tecnomancia. Por que escolher metáfora ao invés de nomes neutros tipo "raw/processed/notes/paths"?*

**Escolha:** nomenclatura inspirada em magia/grimório, formalizada em [[Dicionario de Magia Tecnomante]].

**Consequências:**

- Identidade forte do vault.
- Custo: curva de aprendizado pra leitores externos; vocabulário precisa ser explicado em cada apresentação.

---

### `publish: false` como default; `publish: true` explícito

- **Data:** *preencher*
- **Status:** ativa

**Contexto:** *preencher — opção mais conservadora (default privado) vs default público.*

**Escolha:** templates de Nota, Glosa, Interview Note nascem com `publish: false`. Só MOC nasce com `publish: true`.

**Consequências:**

- Segurança por default — esquecer de marcar não vaza nada.
- Custo: notas evergreen ficam invisíveis no site até alguém promover manualmente. Risco de subaproveitar conteúdo bom.

---

### Skill `/glosa` no fluxo de captura

- **Data:** 2026-04-27
- **Status:** ativa

**Contexto:** *preencher — atrito de "li algo, quero guardar"; risco de Pergaminhos virar buraco negro.*

**Escolha:** skill custom que automatiza WebFetch → fichamento estruturado → cleanup de Pergaminhos. Documentada em [[workflow]] e em `docs/superpowers/specs/2026-04-27-glosa-skill-design.md`.

**Consequências:**

- Captura → destilação em ~30s, não em 30min.
- Custo: depende de Claude Code pra processar; só funciona com artigos web (não PDF/vídeo/podcast).

---

## Como adicionar uma decisão nova

1. Copie o template acima.
2. Preencha contexto, escolha, consequências.
3. Insira na seção "Decisões registradas" em ordem cronológica reversa (mais nova no topo).
4. Se uma decisão antiga foi revisada, mude o `Status` pra `revisada` e adicione uma nova decisão referenciando-a.

## Veja também

- [[Como usar este vault]] — estrutura final que essas decisões produziram
- [[workflow]] — pipeline operacional
- [[Publicação]] — implementação da separação público/apocrypha
