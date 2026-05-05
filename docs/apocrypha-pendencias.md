# Pendências do Apocrypha

Backlog de alterações que precisam ser aplicadas no `codex-technomanticus-apocrypha` quando a próxima sessão dedicada acontecer. Não tocar no apocrypha agora — outra sessão de edição está em andamento.

## Cross-vault awareness

- Configurar o `cta` como **tomo irmão** do `ct`: Claude deve ter conhecimento do `ct` quando estiver operando dentro do `cta` (e vice-versa, na medida do necessário).
- Pontos prováveis de configuração:
  - `CLAUDE.md` no apocrypha referenciando o vault público
  - `additionalDirectories` em settings (já existe parcialmente para `josenaldo.github.io/content/blog` no público)
  - Skills/templates compartilhados ou espelhados
- Decidir: até que ponto o Claude no público pode "saber" do apocrypha? Hoje a regra é total isolamento (memória `feedback_public_apocrypha_separation.md`). A nova regra pode ser: público ignora apocrypha; apocrypha conhece público.

## Tracking de progresso de estudo (spec 2026-05-03)

Parte do design "tracking de progresso nas sendas" mora no apocrypha (vide `docs/superpowers/specs/2026-05-03-progresso-sendas-design.md`). Pendências:

- **Dashboard agregado de progresso** por senda e por domínio (visão consolidada que cruza estado factual do público com narrativa pessoal). Vai pro plano de reestruturação geral do apocrypha.
- **"O que estudar hoje?"** — nota/dashboard alimentada pelo estado de `progresso` cruzado com ordem de estudo. Depende de definirmos como representar ordem global e prioridade.

## Dashboard cross-glosa (spec 2026-05-04)

Da spec `docs/superpowers/specs/2026-05-04-glosa-promocao-design.md` (§13 trabalhos subsequentes):

- **Dashboard cross-glosa**: visão consolidada do repositório de glosas — taxa de promoção, glosas mais antigas em fila, glosas arquivadas por época, top tags, glosas sem promoção há > X dias. Mora no apocrypha porque é leitura analítica/pessoal sobre o próprio processo. Pode reaproveitar Dataview lendo dados do vault público (via `additionalDirectories` ou similar).

## Daily notes do Obsidian

**Decisão fechada:** daily notes vão no apocrypha (cunho pessoal demais pra publicar).

**Proposta detalhada (a refinar na sessão dedicada do apocrypha):**

### Estrutura de pastas

```
Diário/
└── <ano>/
    └── <YYYY-MM-DD>.md
```

`Diário/2026/2026-05-05.md`. Cap natural por ano evita pasta gigante quando o volume crescer.

### Plugin

Plugin core "Daily Notes" do Obsidian (já vem com Obsidian; ativar nas Settings → Core plugins). Configurar:
- **Date format:** `YYYY-MM-DD`
- **New file location:** `Diário/{{date:YYYY}}` (ou só `Diário` se preferir flat — decidir).
- **Template file location:** `00-Meta/templates/Template - Daily.md` (criar este template no apocrypha — não mexer no público).

Plugin Templater já está instalado no público; se for o caso, instalar/ativar no apocrypha também.

Plugin Calendar (community) é opcional — visualização mensal clicável das daily notes. Bom QoL.

### Template proposto (a criar em `Apocrypha/00-Meta/templates/Template - Daily.md`)

```markdown
---
title: "<% tp.date.now('YYYY-MM-DD') %>"
type: daily
created: <% tp.date.now('YYYY-MM-DD') %>
tags:
  - daily
publish: false
---

# <% tp.date.now('YYYY-MM-DD') %>

## Foco do dia

(o que pretende fazer hoje — objetivos prioritários)

## Estudei

(o que estudou; pode incluir wikilinks pra sendas/notas/glosas)

## Worklog

(decisões técnicas, código, problemas resolvidos)

## Capturas

(ideias soltas, links, fragmentos — candidatos a virar pergaminhos/glosas/notas)

## Reflexões

(journal — o que aprendi sobre meu processo, frustrações, retomadas)

## TODOs

- [ ] 

## Para amanhã

(o que ficou pendente; primeira coisa a olhar amanhã)
```

Seções podem ficar vazias — não tem obrigação de preencher tudo.

### Uso múltiplo (não só journal de estudo)

A daily note serve como container do dia, abrigando:

- **Worklog técnico** (decisões, código, debugging).
- **Journal de estudo** (o que aprendi, conexões, dúvidas).
- **Captura rápida** (similar a inbox; vira candidato a pergaminho público depois).
- **Reuniões** (notas de cada uma).
- **TODOs do dia** (checkbox).
- **Reflexões pessoais** (sentimentos, hábitos, observações).

Tudo num arquivo por dia. Se um aspecto crescer demais, pode "promover" pra arquivo dedicado.

### Ciclo de vida

Daily notes ficam **permanentes** — são histórico, não inbox. Não arquivar nem deletar. A subdivisão por ano (`Diário/<ano>/`) já organiza naturalmente.

Possível skill futura: `/daily-resumo-mensal <ano>-<mês>` que sumariza o mês a partir de N daily notes — útil pra retrospectivas.

### Promoção pra outras notas

Daily note funciona como **inbox temporal**:

- Capturou um link interessante → pode ir pra `01-Pergaminhos/entradas.md` (público) e depois virar glosa via `/glosa`.
- Anotou um insight de estudo → pode virar nota em `03-Domínios/<X>/` (público).
- Decisão técnica importante → vira nota dedicada (ADR ou similar).
- Reflexão pessoal → fica na daily mesmo (apocrypha é o lar dela).

A daily permanece como registro do "quando" — não some quando o conteúdo é promovido. Ela é cronológica; as notas promovidas são temáticas. Os dois coexistem.

Skill futura possível: `/daily-promover` que pesca um trecho marcado da daily e move pro destino certo. Mas é overkill pro MVP — manualmente já funciona (copiar/colar + criar arquivo no destino).

### Integração com Codex (público)

A daily note pode citar wikilinks de notas públicas: `[[Senda IA]]`, `[[03-Domínios/React/MUI]]`, `[[02-Glosas/2026-design-md-spec-coding-agents]]`. Backlinks dessas notas mostrariam **quando** elas foram tocadas no journal. Útil pra "que dia trabalhei em X?".

Pra isso, o apocrypha precisa ter acesso de leitura ao público (resolvido pela cross-vault awareness, vide topo deste arquivo).

### Privacidade dentro do apocrypha

Mesmo no apocrypha, pode haver gradação futura:
- Worklog técnico/aprendizado pode eventualmente virar post de blog (`publish: true` numa nota promovida).
- Reflexões pessoais ficam mesmo.

Pro MVP, todas as daily notes têm `publish: false`. Refinar se aparecer caso real.

### Disciplina

Daily note só funciona com hábito. Sugestão: configurar atalho rápido (Ctrl+Alt+D ou similar) e abrir como primeira coisa ao começar a trabalhar. Sem hábito, vira papel de parede.

## Outras pendências

(adicionar aqui conforme forem aparecendo nas sessões do público)
