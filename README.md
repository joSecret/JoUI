# JoUI

Bibliotecă de componente HTML/CSS cu JS minim, fără Tailwind sau alt framework CSS.
Același HTML și CSS funcționează în HTML static, Astro, Drupal și Laravel.

> Proiect la început. Vezi [PLAN.md](PLAN.md) pentru plan și [CONVENTII.md](CONVENTII.md) pentru convenții.

## Structură

```
src/
  css/          bază, tokeni, layout, utilitare
  components/   componente (button, dialog, card…): .css + .md (+ .js)
  sections/     secțiuni din componente (hero, pricing, faq…)
site/           site Astro: starter + demo + documentație
drupal/         temă Drupal cu componente SDC
mcp/            serverul MCP pentru AI
.claude/        comenzi și hook-uri pentru Claude Code
AGENTS.md       regulile pentru orice asistent AI
```

## Principii

- O singură sursă de adevăr: `src/`. Astro și Drupal doar împachetează același HTML/CSS.
- Elemente HTML native înainte de JS.
- Clase după standardul Drupal (BEM), doar tokeni (variabile CSS) pentru valori.
