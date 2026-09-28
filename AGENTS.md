# AGENTS.md: reguli JoUI pentru orice AI

> Sursa unică de instrucțiuni pentru asistenții AI (Claude Code, Cursor, Copilot, Windsurf etc.).
> Într-un site creat din JoUI, `site/PROJECT.md` are prioritate față de acest fișier.

## Ordinea de citire

1. `PROJECT.md` (dacă există): regulile proiectului curent. Au prioritate.
2. `AGENTS.md` (acest fișier): regulile generale JoUI.
3. `CONVENTII.md`: clase CSS, straturi, Drupal, Astro, documentația componentelor.
4. `PLAN.md`: planul și deciziile luate.

## Ce este JoUI

Bibliotecă de componente **HTML/CSS cu JS minim**, fără Tailwind, React sau alt framework. Același HTML și CSS funcționează în HTML static, Astro, Drupal (SDC) și Laravel.

Trei niveluri:
- **Componente** (`src/components/`): primitive (button, dialog, accordion, card…).
- **Secțiuni** (`src/sections/`): blocuri de pagină din componente (hero, pricing, faq…).
- **Pagini** (`site/src/pages/`): pagini-șablon asamblate din secțiuni.

## Reguli obligatorii

1. **Întâi refolosește.** Caută o componentă sau secțiune existentă înainte să creezi una nouă.
2. **O componentă nouă intră în bibliotecă doar după aprobarea proprietarului.**
3. **Elemente native înainte de JS**: `<dialog>`, `<details>`, `popover`, `commandfor`/`command`, formulare native.
4. **Clase după standardul Drupal** (BEM): `.card`, `.card__title`, `.card--featured`, `.is-*` pentru stări, `.js-*` doar pentru JS. Detalii în `CONVENTII.md`.
5. **Doar tokeni** (`--color-*`, `--space-*`, `--radius-*`…) pentru valori vizuale. Nicio culoare sau spațiere fixă în componente.
6. **Fără Tailwind, fără framework CSS, fără dependențe** în `src/`.
7. **Sursa e `src/`.** Astro (`site/`) și Drupal (`drupal/`) doar împachetează același HTML și CSS; nu schimba markup-ul acolo.
8. **Astro fără stiluri scoped** în componente; stilurile vin din `joui.css`.
9. **Accesibil implicit**: HTML semantic, focus vizibil, contrast, `alt` la imagini.

## Structura unei componente

```
src/components/<nume>/
├── <nume>.css   # în @layer components (și state)
├── <nume>.md    # frontmatter cu metadate + documentație + exemple HTML
└── <nume>.js    # doar dacă e necesar
```

## Commit-uri

Fără linii de atribuire AI în commit-uri (fără `Co-Authored-By`).
