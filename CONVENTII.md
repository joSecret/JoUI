# JoUI: convenții pentru clase CSS, Drupal și Astro

> Document de referință pentru oameni și AI. Versiunea 0.1 · 28 septembrie 2026
> Surse: [Drupal CSS architecture](https://www.drupal.org/docs/develop/standards/css/css-architecture-for-drupal-9), [Drupal SDC](https://www.drupal.org/docs/develop/theming-drupal/using-single-directory-components), [Astro styling](https://docs.astro.build/en/guides/styling/).

---

## 1. Principiul

JoUI urmează **standardul de clase din Drupal** (SMACSS + BEM). E cel mai strict dintre platformele noastre, iar Astro și Laravel nu au un standard propriu de clase, deci îl pot folosi fără nicio adaptare. Rezultatul: **același HTML și același CSS** în HTML static, Astro, Drupal și Laravel.

Ce luăm de la oat rămâne: elemente native (`<dialog>`, `<details>`, `popover`, `commandfor`), JS minim, un singur `joui.css` + `joui.js`. Diferența: în loc să stilizăm doar elementele, componentele au clase după standardul Drupal.

---

## 2. Straturile CSS (SMACSS din Drupal → `@layer`)

Drupal împarte CSS-ul în cinci categorii. JoUI le transformă în `@layer`, în aceeași ordine:

```css
@layer tokens, base, layout, components, state, utilities;
```

| Drupal (SMACSS) | JoUI `@layer` | Ce conține | Fișier |
|---|---|---|---|
| Theme (valori) | `tokens` | variabile CSS, `light-dark()` | `src/css/01-tokens.css` |
| Base | `base` | **doar elemente HTML, fără clase** (`h1`, `a`, `button`, `input`, `dialog`) | `src/css/00-base.css` |
| Layout | `layout` | așezarea în pagină: `.layout-container`, `.grid`, `.stack` | `src/css/layout.css` |
| Component | `components` | componentele și secțiunile | `src/components/*/*.css`, `src/sections/*/*.css` |
| State | `state` | `.is-*` și stări native (`[open]`, `:disabled`, `[aria-expanded]`) | în fișierul componentei, în `@layer state` |
| — | `utilities` | puține ajutoare (`.visually-hidden`) | `src/css/utilities.css` |

Așa, stratul `base` face ca HTML-ul simplu (și cel generat de Drupal core) să arate bine din prima, ca la oat.

---

## 3. Numele claselor (BEM, stil Drupal)

```
.component                 bloc          .card
.component__element        sub-element   .card__title, .card__media
.component--variant        variantă      .card--featured
.is-state                  stare         .is-active, .is-open
.js-hook                   pentru JS     .js-tabs   (NICIODATĂ pentru stil)
```

Reguli (din standardul Drupal):

1. **Cuvinte întregi**, nu prescurtări: `.button`, nu `.btn`; `.navigation`, nu `.nav`.
2. **Varianta stă mereu lângă clasa de bază**: `class="card card--featured"`, niciodată doar `card--featured`.
3. **Sub-elementele nu copiază adâncimea DOM-ului**: `.menu__link`, nu `.menu__item__link`.
4. **Fără ID-uri în CSS.** ID-urile sunt doar pentru legături (`commandfor`, `aria-controls`, ancore).
5. **Fără selectori descendenți lungi**; maximum 2 combinatori; preferă copilul direct (`.list > li`).
6. **Componenta nu își controlează poziția în pagină** (fără `margin` exterior, fără `width` fix). De asta se ocupă layout-ul sau părintele.
7. **`.js-*` doar pentru JavaScript**, niciodată în CSS. **`.is-*` doar pentru stări.**
8. **Fără `!important`**, cu excepția stărilor care trebuie să câștige mereu (`:disabled`, eroare).
9. **Doar tokeni** pentru culori, spațieri, raze, umbre, fonturi. Niciodată valori fixe.

Prefix: **fără prefix** (`.card`, nu `.joui-card`), cum face și Drupal. Dacă într-un proiect apare un conflict, prefixul se poate adăuga la build.

### Tokeni

`--<categorie>-<rol>[-<variantă>]`, fără prefix de bibliotecă:

```css
@layer tokens {
  :root {
    color-scheme: light dark;
    --color-surface: light-dark(#fff, #18181b);
    --color-text: light-dark(#09090b, #fafafa);
    --color-primary: light-dark(#1d4ed8, #93c5fd);
    --color-border: light-dark(#d4d4d8, #52525b);
    --space-1: .25rem;  --space-2: .5rem;  --space-3: 1rem;
    --radius-md: .5rem; --radius-lg: .75rem;
    --shadow-lg: 0 10px 30px rgb(0 0 0 / .15);
    --font-body: system-ui, sans-serif;
  }
}
```

---

## 4. Exemplu complet: componenta `card`

**HTML (sursa, identic pe toate platformele):**

```html
<article class="card card--featured">
  <img class="card__media" src="…" alt="…">
  <h3 class="card__title"><a class="card__link" href="…">Titlu</a></h3>
  <p class="card__body">Text scurt.</p>
</article>
```

**CSS (`src/components/card/card.css`):**

```css
@layer components {
  .card {
    display: grid;
    gap: var(--space-2);
    padding: var(--space-3);
    background: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-lg);
  }
  .card__media { border-radius: var(--radius-md); }
  .card--featured { border-color: var(--color-primary); }
}
@layer state {
  .card:has(.card__link:focus-visible) { outline: 2px solid var(--color-primary); }
}
```

---

## 5. Drupal (Single Directory Components)

Disponibile în core din Drupal 10.3.

### Structura

```
drupal/components/card/
├── card.component.yml   # props (JSON Schema) + slots
├── card.twig            # șablonul (în SDC se numește <nume>.twig)
├── card.css             # atașat automat de Drupal
└── card.js              # atașat automat, doar dacă există
```

> Notă: în SDC fișierul se numește `card.twig`. `*.html.twig` rămâne pentru șabloanele obișnuite ale temei (`node--article.html.twig`), care pot *include* componenta.

### `card.component.yml`

```yaml
$schema: https://git.drupalcode.org/project/drupal/-/raw/HEAD/core/assets/schemas/v1/metadata.schema.json
name: Card
status: stable
props:
  type: object
  properties:
    variant:
      type: string
      enum: [default, featured]
      default: default
    url:
      type: string
slots:
  media: { title: Media }
  title: { title: Titlu }
  body:  { title: Text }
```

### `card.twig`

```twig
{% set classes = ['card', variant != 'default' ? 'card--' ~ variant] %}
<article{{ attributes.addClass(classes) }}>
  {% if media %}<div class="card__media">{% block media %}{{ media }}{% endblock %}</div>{% endif %}
  <h3 class="card__title">
    {% if url %}<a class="card__link" href="{{ url }}">{% endif %}
    {% block title %}{{ title }}{% endblock %}
    {% if url %}</a>{% endif %}
  </h3>
  {% block body %}<div class="card__body">{{ body }}</div>{% endblock %}
</article>
```

### Folosire în temă

```twig
{{ include('joui:card', { variant: 'featured', url: url, title: label }, with_context = false) }}
```

Reguli Drupal:
- Numele componentei (`joui:card`) e `tema:componentă`; folderul și fișierele au același nume.
- Mereu `{{ attributes.addClass(...) }}` pe elementul rădăcină, ca Drupal și modulele să poată adăuga clase și atribute.
- Clasele din Twig sunt **exact** cele din HTML-ul sursă.
- CSS-ul nu se rescrie: e copiat (sau importat) din `src/components/card/card.css`.

---

## 6. Astro

Astro nu are un standard de clase, deci folosim **aceleași clase ca în Drupal**. Convențiile Astro pe care le respectăm:

- Fișierul componentei în **PascalCase**: `Card.astro`.
- `interface Props` pentru opțiuni.
- `<slot name="…" />` pentru zonele de conținut.
- `class:list` pentru clasele condiționale.
- **Fără `<style>` scoped în componente.** Stilurile scoped din Astro adaugă `data-astro-cid-*` și ar face CSS-ul diferit de Drupal. Stilurile vin global, din `joui.css` (importat o singură dată în layout).
- Componenta acceptă `class` și alte atribute din exterior (echivalentul lui `attributes` din Drupal).

### `site/src/components/Card.astro`

```astro
---
interface Props {
  variant?: 'default' | 'featured';
  url?: string;
  class?: string;
  [key: string]: unknown;
}
const { variant = 'default', url, class: className, ...attrs } = Astro.props;
---
<article class:list={['card', variant !== 'default' && `card--${variant}`, className]} {...attrs}>
  {Astro.slots.has('media') && <div class="card__media"><slot name="media" /></div>}
  <h3 class="card__title">
    {url ? <a class="card__link" href={url}><slot name="title" /></a> : <slot name="title" />}
  </h3>
  <div class="card__body"><slot name="body" /></div>
</article>
```

### Layout

```astro
---
import '../../../dist/joui.css';
---
```

---

## 7. Tabel de echivalență Astro ↔ Drupal

Aceasta e baza conversiei automate `.astro` → SDC.

| Concept | Astro | Drupal SDC |
|---|---|---|
| Fișiere | `Card.astro` | `card/card.component.yml` + `card/card.twig` |
| Opțiuni | `interface Props` | `props` (JSON Schema) |
| Valoare implicită | `{ variant = 'default' }` | `default: default` |
| Zonă de conținut | `<slot name="body" />` | `slots.body` + `{% block body %}{{ body }}{% endblock %}` |
| Slot opțional | `Astro.slots.has('media')` | `{% if media %}` |
| Clase condiționale | `class:list={[...]}` | `attributes.addClass([...])` |
| Atribute din exterior | `{...attrs}` + `class` | `{{ attributes }}` |
| CSS | global, din `joui.css` | `card.css` (atașat automat) |
| JS | `<script>` cu `.js-*` | `card.js` (atașat automat) cu `.js-*` + `Drupal.behaviors` |
| Folosire | `<Card variant="featured">` | `include('joui:card', {...})` |

---

## 8. Documentația fiecărei componente (ca oat)

Fiecare componentă are un `.md` cu aceeași structură, din care site-ul generează pagina:

1. **Frontmatter**: nume, nivel (component/section), status, versiune, js, tokeni, props, slots.
2. **Descriere**: 1–2 fraze, când se folosește.
3. **Demo HTML**: exemplul de bază + fiecare variantă (bloc ` ```html demo `).
4. **Clase**: tabel cu blocul, elementele, variantele și stările.
5. **Astro**: exemplu de folosire.
6. **Drupal**: exemplu de `include`.
7. **Accesibilitate**: ce e obligatoriu.
8. **Tokeni folosiți.**
