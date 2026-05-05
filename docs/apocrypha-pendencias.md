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

- **Dashboard agregado** de progresso por senda e por domínio (visão consolidada que cruza estado factual do público com narrativa pessoal). Vai pro plano de reestruturação geral do apocrypha.
- **"O que estudar hoje?"** — nota/dashboard alimentada pelo estado de `progresso` cruzado com ordem de estudo. Depende de definirmos como representar ordem global e prioridade.
- **Daily notes para journal de estudo**: ideia aceita, mas o usuário ainda não tem daily notes no pipeline. Vide `docs/perguntas-abertas.md` — bloco "Daily notes do Obsidian". Decidir aqui também: vão pro apocrypha, pro público, ou mistas.

## Outras pendências

(adicionar aqui conforme forem aparecendo nas sessões do público)
