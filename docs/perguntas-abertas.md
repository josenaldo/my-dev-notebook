# Perguntas abertas sobre o modelo do vault

Questões sobre o pipeline e a estrutura do Codex que apareceram em sessões mas ficaram sem resposta. Cada uma vale uma sessão dedicada futura.

## Daily notes do Obsidian

**Contexto:** apareceu durante o brainstorm de tracking de progresso nas sendas (2026-05-03). A ideia de usar daily notes pra journal de estudo (registro de processo de aprendizagem — "por que pausei X", "que dificuldade tive em Y", retrospectivas) ressoou, mas o usuário ainda não tem daily notes integradas ao pipeline.

**Perguntas a resolver:**

- O que são daily notes no contexto do Codex? Qual o papel delas no pipeline?
- Onde moram? Em `01-Pergaminhos/` (são captura inicial), em uma pasta nova `Diário/`, no apocrypha (se for journal pessoal de estudo)?
- Como usar? Templater + plugin de Daily Notes? Estrutura padrão de cada entrada?
- Ciclo de vida: o que acontece com daily notes antigas? Existe processo de "limpeza/arquivamento"?
- Daily note serve só pra journal de estudo, ou tem outras funções (captura geral do dia, registros profissionais, etc.)?
- Conteúdo de uma daily note pode "promover" pra outras notas (semelhante ao pipeline pergaminho → glosa → domínio)? Como?
- Decisão público × apocrypha: daily notes são todas privadas (apocrypha), todas públicas (`publish: true` ou `publish: false` por default), ou mistas conforme o conteúdo de cada uma?

## Glosa → Domínio: como/quando/por quê

**Contexto:** o usuário mencionou (2026-05-03) que ainda não está claro pra ele "como e por que uma glosa vira domínio". O modelo declarado é: pergaminho → glosa → nota no domínio. Mas o critério de promoção, o ato concreto de promoção, e o ponto de retorno (uma glosa pode voltar pra pergaminho? pode ser arquivada?) ainda não estão documentados.

**Perguntas a resolver:**

- Critério de promoção: quando uma glosa "merece" virar nota de domínio? (e.g., depois de N referências cruzadas? quando o tema reaparece em outro contexto? quando o usuário decide?)
- Ato de promoção: a glosa **vira** uma nota nova (movida pra `03-Domínios/`), **gera** uma nota nova (e a glosa permanece como fichamento de origem), ou **é citada por** uma nota nova (relação de referência)?
- Vínculos: nota de domínio referencia a glosa de origem? Glosa fica "trancada" depois de promovida?
- Granularidade: uma glosa pode dar origem a múltiplas notas de domínio? Múltiplas glosas a uma única nota?
- Status `progresso` na glosa vs na nota promovida: como evolui? (e.g., glosa nasce em `andamento`, vira `feito` quando lida; nota de domínio recém-criada nasce em `pendente` ou já em `andamento`?)
- Existe limpeza/arquivamento de glosas antigas que nunca foram promovidas? Indica algo (interesse passageiro, falta de relevância)?
