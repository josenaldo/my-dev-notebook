---
title: "HTML e CSS"
created: 2026-04-01
updated: 2026-04-01
type: concept
status: seedling
tags:
  - javascript
  - frontend
  - web
  - entrevista
publish: false
---

# HTML e CSS

Os fundamentos da web — estrutura (HTML) e apresentação (CSS) que todo fullstack precisa dominar.

## O que é

HTML (HyperText Markup Language) define a estrutura e o conteúdo das páginas web. CSS (Cascading Style Sheets) controla a apresentação visual. Mesmo usando frameworks como React, entender HTML semântico e CSS moderno é essencial para acessibilidade, SEO, e debugging de layouts.

## Como funciona

### HTML Semântico

```html
<!-- Ruim: div soup -->
<div class="header">
  <div class="nav">...</div>
</div>
<div class="content">...</div>

<!-- Bom: semântico -->
<header>
  <nav>...</nav>
</header>
<main>
  <article>
    <section>...</section>
  </article>
  <aside>...</aside>
</main>
<footer>...</footer>
```

**Por que importa:** acessibilidade (screen readers), SEO (crawlers entendem a estrutura), manutenibilidade.

**Tags essenciais:** `header`, `nav`, `main`, `article`, `section`, `aside`, `footer`, `figure`, `figcaption`, `time`, `dialog`.

### CSS Moderno

**Flexbox** — layout unidimensional (linha ou coluna):

```css
.container {
  display: flex;
  justify-content: space-between; /* eixo principal */
  align-items: center;            /* eixo cruzado */
  gap: 1rem;
}
```

**Grid** — layout bidimensional (linhas e colunas):

```css
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr 300px;
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
}
```

**Regra prática:** Flexbox para componentes (navbar, card, form row). Grid para layouts de página.

### Responsividade

```css
/* Mobile-first */
.card { padding: 1rem; }

@media (min-width: 768px) {
  .card { padding: 2rem; display: flex; }
}

/* Container queries (moderno) */
@container (min-width: 400px) {
  .card { flex-direction: row; }
}
```

### CSS Variables (Custom Properties)

```css
:root {
  --color-primary: #2563eb;
  --color-text: #1e293b;
  --spacing-md: 1rem;
  --radius-md: 0.5rem;
}

.button {
  background: var(--color-primary);
  padding: var(--spacing-md);
  border-radius: var(--radius-md);
}
```

### Especificidade e Cascade

```text
!important > inline style > #id > .class > tag > *
   ∞           1000        100    10      1    0
```

**Regra prática:** evitar `!important` e IDs para styling. Usar classes e metodologia (BEM, utility-first).

### Acessibilidade (a11y)

- **`alt` em imagens:** sempre. Decorativas: `alt=""`
- **Labels em forms:** `<label for="email">` ou `aria-label`
- **Contraste:** mínimo 4.5:1 para texto normal (WCAG AA)
- **Keyboard navigation:** todos os interativos devem ser focáveis e operáveis com teclado
- **ARIA:** `role`, `aria-label`, `aria-describedby` quando HTML semântico não basta

## Quando usar

- **HTML/CSS puro:** landing pages, emails, conteúdo estático
- **Flexbox:** componentes de UI (navbars, cards, forms)
- **Grid:** layouts de página, dashboards
- **CSS Modules / CSS-in-JS:** quando usando React (escopo automático)
- **Tailwind/utility-first:** desenvolvimento rápido, consistência de design system

## Armadilhas comuns

- **Div soup:** usar `<div>` para tudo em vez de tags semânticas
- **`!important` everywhere:** sinal de especificidade descontrolada
- **Não testar em mobile:** sempre começar mobile-first
- **Ignorar acessibilidade:** não é opcional — é requisito legal em muitos mercados
- **Unidades fixas (px):** preferir `rem`, `em`, `%`, `vw/vh` para responsividade

## Na prática (da minha experiência)

> Em projetos React, uso CSS Modules ou styled-components para escopo de estilos. No MedEspecialista, o frontend usa MUI (Material UI) como base, mas customizo com CSS variables para manter o design system consistente. Entender Flexbox e Grid profundamente me permite debugar layouts complexos rapidamente — problemas que parecem bugs de componente geralmente são CSS mal aplicado.

## How to explain in English

"HTML and CSS are the foundation of everything we build on the web. Even when using React or other frameworks, understanding semantic HTML is crucial for accessibility and SEO. I write markup that screen readers can navigate and search engines can index.

For layouts, I use Flexbox for component-level alignment and Grid for page-level structure. The combination covers virtually any layout need. I follow a mobile-first approach with CSS, starting with the smallest viewport and adding complexity through media queries.

Accessibility is something I take seriously. I ensure proper heading hierarchy, meaningful alt text, sufficient color contrast, and full keyboard navigability. It's not just about compliance — it makes the application better for everyone."

### Key vocabulary

- semântica → semantic HTML: usar tags com significado
- acessibilidade → accessibility (a11y)
- responsivo → responsive design
- especificidade → specificity: regra de prioridade CSS
- cascata → cascade: herança de estilos
- mobile-first → mobile-first: design começando pelo menor viewport

## Recursos

- [MDN Web Docs — HTML](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [MDN Web Docs — CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [CSS Tricks — Flexbox Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [CSS Tricks — Grid Guide](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [WCAG Guidelines](https://www.w3.org/WAI/standards-guidelines/wcag/)

## Veja também

- [[JavaScript Fundamentals]]
- [[React]]
- [[Bootstrap]]
- [[Material UI]]
- [[Mantine]]
