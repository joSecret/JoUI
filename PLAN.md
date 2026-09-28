# JoUI: plan de lucru (bibliotecă + documentație + server MCP)

> Document viu. Îl completăm împreună pe parcurs. Bifează `[x]` ce e gata.
> Versiunea planului: 0.6 (structura v2 + clase după standardul Drupal, de confirmat) · 28 septembrie 2026 (fără tokens.json; fluxul design → site Astro → Drupal)

---

## 0. Ideea de bază

**O singură sursă de adevăr.** Componentele și tokenii stau într-un singur loc (repo-ul JoUI). Tot restul doar *citește* de acolo:

```
                ┌─────────────────────────────┐
                │  Repo JoUI (sursa)          │
                │  tokens/  components/       │
                │  GUIDELINES.md              │
                └──────────────┬──────────────┘
                               │ build
                               ▼
                        dist/joui.json  (catalogul complet)
            ┌──────────────────┼───────────────────┐
            ▼                  ▼                   ▼
   Site documentație     Server MCP (AI)     Teme Drupal / Astro
   (Astro, GH Pages)     (npm, stdio)        (folosesc HTML/CSS de bază)
```

Regula de aur: **nu modifici niciodată ceva „în aval”** (în site, în MCP, într-o temă) dacă poate fi modificat în sursă. Dacă schimbi sursa, totul se regenerează.

Ce știm deja despre JoUI:
- Bibliotecă HTML/CSS, cu JS minim (folosește elemente native, de ex. `<details>`).
- Fiecare componentă are un singur HTML/CSS de bază, deja folosit în Drupal și Astro.
- Tokenii sunt variabile CSS, într-un singur loc; dacă îi schimbi local, se restilizează tot site-ul.
- Mai târziu vor fi și alți utilizatori (deci documentația contează).
- Scopul MCP-ului: AI-ul să genereze componente Drupal (SDC) și Astro din imagine / SVG / cod, respectând regulile JoUI.

Ordinea pașilor contează: **fiecare pas se sprijină pe cel dinainte.** MCP-ul vine ultimul pentru că doar „servește” ce ai construit deja.

---

## Pasul 1. Structura repo-ului

**Scop:** oricine (om sau AI) să găsească orice componentă în același fel, fără să ghicească.

### 1.1 Structura propusă (v2: după astrodeck + oat)

> Propunere, de confirmat. Ia de la **astrodeck** organizarea pe trei niveluri și partea pentru AI, și de la **oat** felul în care sunt făcute componentele (HTML semantic, CSS pe elemente native, JS minim). Fără Tailwind, React sau Radix (astrodeck le folosește, noi nu).

```
joui/
├── AGENTS.md              # regulile pentru ORICE AI (Claude, Cursor, Copilot…): sursa unică
├── CLAUDE.md              # o linie: „citește AGENTS.md”
├── README.md · PLAN.md · CHANGELOG.md · LICENSE
├── package.json           # scripturi: build, validate
│
├── src/                   # NUCLEUL, ca oat: HTML/CSS/JS simplu, zero dependențe
│   ├── css/
│   │   ├── 00-base.css    # reset + stiluri pe elemente native (h1…, a, button, input…)
│   │   ├── 01-tokens.css  # toate variabilele, cu light-dark() pentru dark mode
│   │   └── utilities.css  # puține clase utilitare (stack, grid…)
│   ├── components/        # NIVELUL 1: primitive (button, dialog, accordion, tabs, card…)
│   │   └── dialog/
│   │       ├── dialog.css # stilul, în @layer components
│   │       ├── dialog.md  # documentație + exemple HTML (demo) + metadate în frontmatter
│   │       └── dialog.js  # DOAR dacă e nevoie (web component mic)
│   └── sections/          # NIVELUL 2: secțiuni din componente (hero, pricing, faq, footer…)
│       └── faq/
│           ├── faq.css
│           └── faq.md
│
├── dist/                  # generat: joui.css + joui.js (un singur <link> și <script>)
│                          # → merge oriunde: HTML static, Laravel, Drupal, Astro
│
├── site/                  # NIVELUL 3, ca astrodeck: starter Astro + demo + documentație
│   ├── src/components/    # .astro peste nucleu (Dialog.astro, Faq.astro…)
│   ├── src/layouts/
│   ├── src/pages/         # pagini-șablon (landing, despre, contact…) + paginile de documentație
│   └── PROJECT.md         # suprascrierile unui site anume (brand, ton, reguli)
│
├── drupal/                # temă Drupal cu componente SDC (.component.yml + .html.twig)
├── mcp/                   # serverul MCP
│
├── registry.json          # generat: catalogul pentru AI și MCP (ca astrodeck)
├── public/llms.txt        # generat: documentația citibilă de AI
│
└── .claude/
    ├── commands/          # /new-component, /new-section, /theme, /to-drupal
    └── hooks/             # gardian: blochează culori fixe în afara tokenilor, Tailwind etc.
```

Ce se schimbă față de v1:
- **Trei niveluri** (componente → secțiuni → pagini), ca la astrodeck. Pentru fluxul „design → site” contează mult: AI-ul potrivește întâi secțiuni întregi (hero, pricing), nu doar butoane.
- **Site-ul Astro e și starter, și documentație** (ca astrodeck). `create_site` copiază folderul `site/`. Nu mai e nevoie de `apps/docs` și `starters/` separate.
- **Componentele ca oat** (elemente native, JS minim), dar **cu clase după standardul Drupal** (BEM). Stratul `base` stilizează elementele simple fără clase; componentele au clase. Același HTML merge identic în Astro, Drupal și Laravel.
- **Mai puține fișiere per componentă**: `.css` + `.md` (+ `.js` opțional). Exemplele HTML stau în `.md`, iar metadatele (props, slots, tokeni) în frontmatter-ul lui `.md`. Fără `.html` și `.json` separate.
- **`AGENTS.md` + `PROJECT.md`** (ca astrodeck): reguli comune pentru orice AI + suprascrieri pe site. `CLAUDE.md` doar trimite la `AGENTS.md`.
- **`dist/joui.css` + `joui.js`**: un singur fișier de inclus, ca la oat.

### 1.2 Cum arată o componentă (exemplu: dialog)

`src/components/dialog/dialog.md`:

````markdown
---
name: dialog
title: Dialog
tier: component          # component | section
status: stable
version: 1.0.0
js: none                 # none | progressive | required
tokens: [--color-surface, --color-border, --radius-lg, --shadow-lg]
slots:
  header: { required: false }
  body:   { required: true }
  footer: { required: false }
props:
  id:       { type: string, required: true }
  closedby: { type: enum, values: [any, closerequest, none], default: any }
---

Dialog modal pe `<dialog>` nativ. Se deschide cu `commandfor` + `command="show-modal"`, fără JS.

```html demo
<button commandfor="d1" command="show-modal">Deschide</button>
<dialog class="dialog" id="d1" closedby="any">
  <form method="dialog">
    <header><h3>Titlu</h3></header>
    <div><p>Conținut.</p></div>
    <footer><button value="ok">OK</button></footer>
  </form>
</dialog>
```

## Accesibilitate
- Focus-ul, Escape și fundalul funcționează nativ.
````

`src/components/dialog/dialog.css`:

```css
@layer components {
  .dialog {
    background: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow-lg);
  }
  .dialog::backdrop { background: rgb(0 0 0 / .5); }
}
```

### 1.3 Convenții

Detaliate în [`CONVENTII.md`](CONVENTII.md): clase după standardul Drupal (SMACSS + BEM: `.card`, `.card__title`, `.card--featured`, `.is-*`, `.js-*`), `@layer tokens, base, layout, components, state, utilities`, tokeni `--color-*`, `--space-*`, echivalența Astro ↔ Drupal SDC și structura documentației fiecărei componente.

### 1.4 Checklist Pasul 1

- [ ] Creez repo-ul (sau reorganizez pe cel existent) după structura de mai sus.
- [ ] Mut tokenii existenți în `src/css/01-tokens.css` (`--color-*`, `--space-*`…).
- [ ] Grupez tokenii în `tokens.css` pe categorii, cu un comentariu deasupra fiecărui grup (`/* @category color */`); scriptul de build citește categoriile din comentarii.
- [ ] Aleg **3–4 componente reale** pe care le am deja (recomandare: `button`, `card`, `accordion` cu `<details>`, `nav`).
- [ ] Pentru fiecare: `.html`, `.css`, `.md`, `.json`.
- [ ] Scriu `schema/component.schema.json` după modelul de mai sus.
- [ ] Nu trec mai departe până nu arată toate cele 3–4 componente la fel.

---

## Pasul 2. Regulile scrise (`GUIDELINES.md`)

**Scop:** logica ta de proiectare, scrisă o singură dată, pe care o citesc și oamenii, și AI-ul. E documentul cel mai important din tot proiectul: MCP-ul îl dă AI-ului exact așa cum e.

### 2.1 Ce conține (capitole propuse)

1. **Principii** (5–7 fraze scurte). Exemple:
   - HTML semantic înainte de orice.
   - Elemente native înainte de JS (`<details>`, `<dialog>`, `<form>`, `popover`).
   - Doar tokeni, niciodată valori fixe (culori, spațieri, raze, fonturi).
   - Mobil întâi; layout cu `flex`/`grid`, fără lățimi fixe.
   - Accesibil implicit (contrast, focus vizibil, ordine logică).
2. **Tokeni**: categoriile, cum se numesc, când creezi un token nou și când nu.
3. **HTML**: elemente permise/preferate, headings, linkuri vs. butoane, imagini (`alt`, `loading`).
4. **CSS**: BEM după standardul Drupal (vezi CONVENTII.md), fără `!important`, fără ID-uri, specificitate mică, `@layer` (dacă îl folosești), media/container queries.
5. **JS**: când e permis, cum se adaugă progresiv.
6. **Accesibilitate**: minimul obligatoriu pentru orice componentă.
7. **Platforme**:
   - Drupal SDC: cum se transformă `slots`/`props` în `*.component.yml` + Twig.
   - Astro: cum se transformă în `.astro` cu `Astro.props` și `<slot>`.
8. **Cum adaugi o componentă nouă** (pașii, în ordine).
9. **Exemple bune / greșite** (scurte, cu cod). AI-ul învață cel mai mult de aici.

### 2.2 Cum scrii regulile ca să le înțeleagă și AI-ul

- Scurt și imperativ: „Folosește `<button>` pentru acțiuni, `<a>` pentru navigare.”
- Fiecare regulă cu **de ce** într-o jumătate de frază. AI-ul aplică mai bine o regulă dacă știe motivul.
- O regulă = un rând. Evită paragrafele lungi.
- Pune exemple *greșit / corect* una lângă alta.
- Numerotează regulile importante (`R1`, `R2`, …) ca să le poți cita în verificări („încalcă R4”).

### 2.3 Checklist Pasul 2

- [ ] Scriu principiile (5–7).
- [ ] Scriu regulile pentru tokeni, HTML, CSS, JS, accesibilitate.
- [ ] Scriu secțiunea „Platforme” (după ce termin Pasul 4 o actualizez).
- [ ] Adaug cel puțin 5 exemple „greșit / corect”.
- [ ] Pun `version` în capul fișierului (de ex. `Guidelines v1.0`).
- [ ] Verific: cele 3–4 componente din Pasul 1 respectă toate regulile? Dacă nu, corectez componenta **sau** regula.

---

## Pasul 3. Site-ul de documentație

**Scop:** o vitrină pentru oameni, generată automat din Pasul 1, și locul de unde se publică `joui.json`.

### 3.1 Alegeri recomandate

- **Inspirație: tema Compass** (MIT, gratuită): structura ei (navigație pe categorii, căutare Pagefind, dark mode, breadcrumbs) e bună. Dar Compass folosește **Tailwind 4**, așa că nu o luăm ca atare: îi păstrăm structura Astro și logica, și înlocuim tot stilul cu JoUI. Așa site-ul de documentație e și prima dovadă că JoUI merge.

- **Astro** (îl folosești deja) sau **Starlight** (tema de documentație pentru Astro, gata cu căutare, navigație, dark mode).
- Găzduire: **GitHub Pages** (gratuit, se publică automat la fiecare push pe `main`).
- Site-ul **nu are conținut propriu despre componente**: citește `components/*/` și generează paginile.

### 3.2 Ce pagini are

- **Start**: ce e JoUI, instalare, cum folosești tokenii.
- **Tokeni**: tabel generat direct din `tokens.css`, cu mostre de culoare/spațiere; opțional un editor live care îți arată cum arată site-ul cu alți tokeni.
- **Componente**: câte o pagină per componentă, generată din:
  - `card.md` → textul;
  - `card.html` + `card.css` → demo live;
  - `card.json` → tabelul de slots/props/tokeni, statusul, platformele;
  - cod de copiat pentru HTML, Drupal și Astro.
- **Ghid**: `GUIDELINES.md` afișat ca pagină.
- **Changelog**.

### 3.3 Catalogul `joui.json`

La fiecare build, `scripts/build-catalog.mjs` produce `dist/joui.json`, care se publică și pe site (de ex. `https://<user>.github.io/joui/joui.json`):

```json
{
  "name": "joui",
  "version": "0.3.0",
  "guidelinesVersion": "1.0",
  "generatedAt": "2026-10-01T10:00:00Z",
  "tokens": [ { "name": "--color-surface", "value": "#fff", "category": "color" } ],
  "components": [ { "...": "conținutul fiecărui card.json + html + css + md" } ],
  "guidelines": "…textul complet din GUIDELINES.md…"
}
```

Acesta e „contractul” dintre bibliotecă și MCP. **MCP-ul citește doar acest fișier.**

### 3.4 Checklist Pasul 3

- [ ] Creez site-ul în `docs/` (Astro sau Starlight).
- [ ] Pagina de tokeni, generată din `tokens.css`.
- [ ] Pagina de componentă, generată din cele 4 fișiere.
- [ ] Scriptul `build-catalog.mjs` → `dist/joui.json`.
- [ ] GitHub Action: la push pe `main` → validare → build → publicare pe GitHub Pages.
- [ ] Verific: adaug o componentă nouă (doar cele 4 fișiere) → apare singură pe site.

---

## Pasul 4. Șabloanele de platformă (exemplele-model)

**Scop:** 2–3 componente făcute *de mână*, perfect, în Drupal și Astro. Ele devin modelul după care AI-ul le face pe restul. Un exemplu bun valorează mai mult decât zece reguli.

### 4.1 Ce alegi ca model

Alege componente diferite ca tip, ca să acopere cazurile:
1. **Simplă, fără slots complexe**: `button` (doar props).
2. **Cu slots**: `card` (media, titlu, text, acțiuni).
3. **Cu element nativ interactiv**: `accordion` pe `<details>`.

### 4.2 Drupal (Single Directory Components)

```
packages/drupal/card/
├── card.component.yml   # props + slots, generate din card.json
├── card.twig            # markup-ul din card.html, cu variabile Twig
└── card.css             # același CSS (sau import din sursă)
```

Reguli de urmat:
- Twig-ul păstrează **exact** clasele și structura din `card.html`.
- `props` și `slots` din `.component.yml` au aceleași nume ca în `card.json`.
- CSS-ul nu se rescrie: e cel din `components/card/card.css`.

### 4.3 Astro

```
packages/astro/Card.astro
```

- `interface Props` generat din `props` din `card.json`.
- `<slot name="media" />` etc. pentru fiecare slot.
- Același markup și aceleași clase ca în `card.html`.

### 4.4 Checklist Pasul 4

- [ ] `button`, `card`, `accordion` în Drupal SDC, testate într-un site Drupal real.
- [ ] Aceleași trei în Astro, testate într-un site Astro real.
- [ ] Actualizez `platforms` în fiecare `.json` (`status: "done"`).
- [ ] Scriu în `GUIDELINES.md` → „Platforme” cum s-a făcut transformarea (pașii exacți).
- [ ] Notez ce a fost greu sau ambiguu → devine regulă nouă în ghid.

---

## Pasul 5. Serverul MCP

**Scop:** AI-ul (Claude, Cursor, VS Code etc.) să aibă acces direct la JoUI și să genereze componente corecte din imagine, SVG sau cod.

### 5.1 Cum funcționează

- Un pachet npm mic, de ex. `@jocode/joui-mcp`, pornit local prin **stdio** (`npx @jocode/joui-mcp`).
- Scris în TypeScript cu SDK-ul oficial `@modelcontextprotocol/sdk`.
- La pornire încarcă `joui.json` (din pachet sau de pe URL-ul site-ului, ca să fie mereu la zi).
- **Nu conține reguli proprii.** Tot ce știe vine din `joui.json` și `GUIDELINES.md`.

### 5.2 Fluxul principal: design → site Astro → Drupal

Regula de bază: **întâi se refolosește, abia apoi se creează.** Un site nou pornește mereu cu toate componentele JoUI existente.

1. **Pornire.** Utilizatorul dă un design din orice sursă (imagine, Figma, SVG, cod, link). MCP-ul creează un site Astro nou în care **copiază** toate componentele JoUI (`create_site`). Fiind copii, în site le poți modifica liber, fără să atingi biblioteca.
2. **Potrivire.** AI-ul analizează designul și, pentru fiecare bucată, caută în bibliotecă (`match_components`): „header” → `nav`, „box cu poză” → `card`, „întrebări” → `accordion`. Rezultatul e o listă: *există* / *există cu altă variantă* / *lipsește*.
3. **Tokeni.** AI-ul extrage din design culorile, fonturile, spațierile, razele și scrie un fișier de tokeni al site-ului (`generate_site_tokens`), de ex. `src/styles/site-tokens.css`, care suprascrie doar valorile din `tokens.css`. Componentele nu se ating: site-ul arată ca designul doar din tokeni.
4. **Ce lipsește.** Doar pentru bucățile care nu există, AI-ul creează componente noi după ghid și după modelele din Pasul 4, apoi le verifică (`validate_component`).
5. **Revizuire.** Tu te uiți pe site-ul Astro. Ajustezi tokenii sau componentele până e aproape de design.
6. **Drupal.** Când site-ul Astro e gata, fiecare componentă `.astro` se transformă în Drupal SDC (`convert_to_drupal`): `Card.astro` → `card.component.yml` + `card.html.twig` (+ același CSS). Transformarea e aproape mecanică pentru că `props` și `slots` sunt deja descrise în `.json`-ul componentei și markup-ul e același.
7. **Înapoi în bibliotecă, doar cu aprobare.** Componentele noi (sau modificările bune făcute într-un site) intră în JoUI numai după ce le aprobi tu, ca propunere (PR) cu cele 4 fișiere. Următorul site le găsește deja la pasul 2.

> Pentru că site-urile au copii, fiecare componentă copiată păstrează în `.json` câmpul `version` din bibliotecă. Așa poți vedea mai târziu dacă un site are o versiune veche și ce s-a schimbat de atunci (în `CHANGELOG.md`).

De ce merge conversia ușor: în Astro și în Drupal **HTML-ul și CSS-ul sunt identice**; diferă doar „ambalajul” (`Astro.props` + `<slot>` vs. `props`/`slots` în `.yml` + variabile Twig). Dacă nu se schimbă markup-ul, conversia nu poate strica aspectul.

### 5.3 Ce oferă AI-ului

**Tools** (acțiuni pe care AI-ul le poate cere):

| Tool | Ce face |
|---|---|
| `list_components` | Lista componentelor (nume, categorie, status, descriere). |
| `get_component` | Totul despre o componentă: HTML, CSS, md, slots, props, tokeni, variantele Astro și Drupal. |
| `match_components` | Primește o descriere a bucăților din design și răspunde, pentru fiecare, cu componenta JoUI potrivită (sau „lipsește”). Oprește AI-ul să recreeze ce există deja. |
| `get_tokens` | Tokenii din `tokens.css`, pe categorii. |
| `generate_site_tokens` | Primește valorile extrase din design și scrie fișierul de tokeni al site-ului (doar suprascrieri, doar tokeni existenți). |
| `create_site` | Creează proiectul Astro de start cu toate componentele JoUI și tokenii de bază. |
| `get_guidelines` | Ghidul complet sau o secțiune. |
| `get_platform_template` | Modelul pentru `astro` sau `drupal`. |
| `validate_component` | Verifică HTML/CSS generat după reguli (clase BEM după standardul Drupal, doar tokeni, fără ID-uri, `alt` la imagini…) și spune ce regulă e încălcată. |
| `convert_to_drupal` | Din `.astro` (+ `.json`) produce `.component.yml` + `.html.twig`. |

**Resources:** `joui://guidelines`, `joui://tokens`, `joui://components/{name}`.

**Prompts:** `design-to-site` (tot fluxul de mai sus, pas cu pas), `convert-site-to-drupal`.

### 5.4 Checklist Pasul 5

- [ ] Creez `mcp/` cu TypeScript + `@modelcontextprotocol/sdk`.
- [ ] Implementez întâi doar tools-urile de citire: `list_components`, `get_component`, `get_guidelines`, `get_tokens`, `match_components`.
- [ ] Testez cu **MCP Inspector** (`npx @modelcontextprotocol/inspector`).
- [ ] Îl conectez în Claude Desktop / Claude Code și încerc fluxul pe o imagine reală.
- [ ] Adaug `validate_component`, `get_platform_template`, `generate_site_tokens`.
- [ ] Adaug `create_site` și `convert_to_drupal`; testez tot fluxul pe un design real.
- [ ] Adaug prompts.
- [ ] Public pe npm; scriu în README cum se configurează.

---

## 6. Cum schimbi pe parcurs fără să strici nimic

### 6.1 Versionare

- **Biblioteca** folosește SemVer în `package.json`:
  - `patch` (0.3.1): corecturi de stil fără schimbare de markup;
  - `minor` (0.4.0): componentă nouă, variantă nouă, token nou;
  - `major` (1.0.0): schimbi markup, redenumești clase sau tokeni (strică temele existente).
- **Fiecare componentă** are `version` în `.json`.
- **Ghidul** are versiunea lui (`Guidelines v1.1`), notată și în `joui.json`.
- `CHANGELOG.md` la fiecare release.

### 6.2 Unde se schimbă ce

| Vreau să schimb… | Schimb în… | Nu schimb în… |
|---|---|---|
| Aspectul global | `tokens/tokens.css` | CSS-ul componentelor |
| O componentă | `components/<nume>/` | site, MCP, teme |
| O regulă | `GUIDELINES.md` (+ exemplele) | MCP |
| Cum se face în Drupal/Astro | `platforms/` + secțiunea „Platforme” | MCP |
| Ce poate face AI-ul | `mcp/` (tools noi) | datele |

### 6.3 Structura în sine

Dacă schimbi structura (de ex. adaugi un câmp nou în `card.json`):
1. Actualizezi `schema/component.schema.json`.
2. Rulezi `validate.mjs` → îți arată toate componentele care trebuie actualizate.
3. Le actualizezi pe toate (sau câmpul are o valoare implicită).
4. Crești versiunea minoră.

### 6.4 Ritm de revizuire

- La fiecare **5–10 componente noi**: recitești ghidul, adaugi reguli pentru ce a apărut repetat, ștergi ce nu mai e adevărat.
- La fiecare componentă generată de AI care a ieșit greșit: întrebi „ce regulă lipsea?” și o adaugi. Așa ghidul se îmbunătățește singur în timp.

### 6.5 Automatizări (GitHub Actions)

- La fiecare PR: `validate.mjs` (structură + reguli) → eșuează dacă lipsește un fișier sau un câmp.
- La push pe `main`: build site + `joui.json` → GitHub Pages.
- La tag `v*`: publicare pachet CSS și pachet MCP pe npm.

---

## 7. Ordinea concretă (ce faci săptămâna asta)

1. [ ] **Ziua 1–2:** Pasul 1 cu 3–4 componente reale + `tokens.css`.
2. [ ] **Ziua 3:** Prima versiune de `GUIDELINES.md` (principii + reguli + 5 exemple).
3. [ ] **Ziua 4:** `schema/component.schema.json` + `validate.mjs` + `build-catalog.mjs`.
4. [ ] **Săptămâna 2:** Pasul 4 (modelele Drupal + Astro) pe `button`, `card`, `accordion`.
5. [ ] **Săptămâna 2–3:** Pasul 3 (site-ul), minim: pagină de tokeni + pagină de componentă.
6. [ ] **Săptămâna 3:** Pasul 5, MCP-ul minim (4 tools), testat pe o imagine reală.

> De ce Pasul 4 înaintea Pasului 3? Site-ul arată și codul Drupal/Astro; dacă ai modelele înainte, site-ul le afișează din prima. Dacă vrei site-ul mai repede, le poți inversa.

---

## 8. Decizii luate

- `tokens.json` nu există; tokenii se citesc direct din `tokens.css`.
- Fluxul principal: design → site Astro cu toate componentele → potrivire (refolosire întâi) → tokenii site-ului → doar componentele lipsă → conversie în Drupal SDC.
- Componentele se **copiază** în site-ul nou și se pot modifica liber acolo.
- O componentă nouă intră în bibliotecă **doar după aprobarea ta**.
- Designul poate veni din orice sursă.
- Fără Tailwind sau alt framework CSS: nucleu HTML/CSS/JS simplu, portabil (și pe Laravel), apoi componente Astro și Drupal.
- Un singur repo (monorepo) cu `packages/core`, `astro`, `drupal`, `mcp` și `apps/docs`.
- Site-ul de documentație pornește de la structura temei Compass, restilizată cu JoUI.

## 8.1 Întrebări deschise (le completăm împreună)

- [ ] Repo-ul JoUI există deja pe GitHub? Care e numele (`owner/repo`)?
- [ ] Câte componente ai deja și care sunt?
- [ ] Folosești `@layer` în CSS? Ai deja o convenție de nume pentru clase?
- [ ] Vrei să publici JoUI pe npm (ca pachet CSS) sau doar ca repo?
- [ ] Documentația: Astro simplu sau Starlight?
- [ ] Ce versiune de Drupal (10/11) și ce temă de bază folosești?
- [ ] Ce AI vei folosi cel mai des cu MCP-ul (Claude Desktop, Claude Code, Cursor, VS Code)?

---

## 9. Notițe

_(spațiu liber pentru decizii și idei pe parcurs)_
