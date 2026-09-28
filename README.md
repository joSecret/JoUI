# JoUI

Bibliotecă de componente HTML/CSS cu JS minim, fără Tailwind sau alt framework CSS.
Nucleul e portabil (HTML static, Astro, Drupal, Laravel); peste el vin componentele pentru fiecare platformă.

> Proiect la început. Planul complet e în [PLAN.md](PLAN.md).

## Structură

```
packages/
  core/        sursa: tokeni, stiluri de bază, componente HTML/CSS/JS
  astro/       componentele .astro
  drupal/      componentele Drupal SDC (.component.yml + .html.twig)
  mcp/         serverul MCP pentru AI
apps/
  docs/        site-ul de documentație (Astro, construit cu JoUI)
starters/
  astro-site/  șablonul pentru un site nou
schema/        schema pentru metadatele componentelor
scripts/       validare și build pentru catalog
```

## Principii

- O singură sursă de adevăr: componentele și tokenii stau în `packages/core`, restul doar citește de acolo.
- HTML semantic și elemente native înainte de JS.
- Doar tokeni (variabile CSS), niciodată valori fixe.
