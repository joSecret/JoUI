# CLAUDE.md

Instrucțiuni pentru Claude când lucrează în acest repo.

- Răspunde și scrie documentația în română.
- Citește întâi `PLAN.md` (planul și deciziile luate) și `GUIDELINES.md` (regulile, când există).
- Fără Tailwind sau alt framework CSS. Nucleul (`packages/core`) e HTML/CSS/JS simplu, fără dependențe.
- Sursa e `packages/core`. Nu schimba o componentă direct în `astro/`, `drupal/`, `mcp/` sau `apps/docs` dacă schimbarea ține de sursă.
- Fiecare componentă din `packages/core/components/<nume>/` are exact aceleași fișiere: `.html`, `.css`, `.md`, `.json`.
- Doar tokeni `--joui-*` în CSS; clase cu prefix `joui-` și BEM ușor.
- Întâi refolosește o componentă existentă, abia apoi creează una nouă. O componentă nouă intră în bibliotecă doar după aprobarea proprietarului.
- Commit-urile nu conțin linii de atribuire Claude (fără `Co-Authored-By: Claude`).
