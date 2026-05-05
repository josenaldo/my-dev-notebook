---
title: "Publicação"
created: 2026-04-28
updated: 2026-04-28
type: how-to
status: seedling
tags:
  - guia
  - meta
publish: false
---

# Publicação

Como o conteúdo deste vault é publicado no site público e — mais importante — como **garantir que conteúdo privado nunca vaza**.

## Arquitetura de 3 repos

```text
codex-technomanticus              (público)   ← este vault, é o que você edita
codex-technomanticus-site         (público)   ← Quartz v4, faz o build
codex-technomanticus-apocrypha    (privado)   ← conteúdo sensível, nunca cruza com site
```

A separação é **física**: apocrypha vive em outro repositório. Não há link, símbolo ou git submodule entre o público e o apocrypha.

## Pipeline de deploy

```text
push em codex-technomanticus (main)
  └→ workflow Trigger Site Deploy
       └→ repository_dispatch event-type: content-update
            └→ site repo: workflow Deploy to GitHub Pages
                 ├─ checkout do site repo (Quartz)
                 ├─ checkout do codex-technomanticus em content/
                 ├─ rename README.md → index.md em cada pasta
                 ├─ npx quartz build
                 └─ deploy GitHub Pages
```

Implementação:

- Trigger: `.github/workflows/trigger-site-deploy.yaml` no repo público.
- Build: `.github/workflows/deploy.yaml` no site repo.
- O apocrypha **não é checkout em nenhum momento** do build — sequer chega na máquina que builda.

## O que é publicado

Quartz aplica o filtro `Plugin.RemoveDrafts()`. Uma nota é publicada quando:

- `publish: true` (ou ausência do campo, que é tratado como `true` pelo Quartz)
- E não tem `draft: true`

Uma nota é **suprimida** quando:

- `publish: false` no frontmatter
- Está em pasta `private/` (via `ignorePatterns` no `quartz.config.ts`)
- Tem `draft: true`

Por convenção neste vault:

| Tipo de nota         | `publish` default |
| -------------------- | ----------------- |
| MOC                  | `true`            |
| Domínio (`evergreen`)| `true` (caso a caso)             |
| Domínio (`seedling`) | `false`           |
| Glosa                | `false`           |
| Senda                | depende — só publique sendas curadas |
| Notas em `00-Meta/`  | só `README.md` e o que for explicitamente `publish: true` |

## A regra de ouro: zero referência a apocrypha

**Nenhum arquivo público pode referenciar conteúdo do apocrypha** — nem wikilinks, nem paths, nem menções no corpo.

Por quê? Mesmo que o arquivo apocrypha não exista neste repo, um wikilink como `[[Algum Tema Privado]]`:

- Vaza o **título** do que é privado pra qualquer um que ler o markdown
- Aparece como link não-resolvido no site público (vermelho), chamando atenção
- Se algum dia esse título virar uma nota pública por engano, o link silenciosamente conecta privado com público

A separação tem que ser **invisível** — o público não pode nem suspeitar que existe um apocrypha.

### Checklist anti-vazamento (antes de cada push grande)

- [ ] Nenhum wikilink aponta pra nota inexistente que tem nome de tema privado
- [ ] Nenhum path com `apocrypha` em arquivo público
- [ ] Nenhuma menção a clientes, projetos pagos, dados de pessoas em arquivos públicos (vide regra geral de não fabricar/expor terceiros)
- [ ] Notas com dados sensíveis estão com `publish: false` **e** o motivo está claro pra você

### Detectar referências a apocrypha

```bash
# rodar na raiz do repo público
grep -rn "apocrypha" --include="*.md" .
```

Se a busca retornar qualquer coisa em arquivo `publish: true`, é vazamento.

## README.md → index.md

O deploy renomeia todo `README.md` pra `index.md` antes do build do Quartz. Isso significa:

- O `README.md` raiz do repo é também a landing page pública.
- Cada pasta com `README.md` ganha um índice automático no site.
- `ignorePatterns` no Quartz ignora `**/README.md` — necessário porque o índice já é o `index.md` gerado.
- **Não delete o `index.md` raiz** — Quartz v4 quebra a landing sem ele. (Lembrete: este projeto coexiste `index.md` e `README.md` no root.)

## Quando fizer mudanças no pipeline

Mudou algo que afeta build/deploy? Documente em [[Decisões do vault]]. Mudanças sensíveis: `quartz.config.ts`, `ignorePatterns`, lista de plugins, workflow YAMLs.

## Veja também

- [[Convenções de escrita]] — campo `publish` no frontmatter
- [[Decisões do vault]] — registros de decisões de arquitetura
- [[workflow]] — pipeline interno (não confundir com pipeline de deploy)
