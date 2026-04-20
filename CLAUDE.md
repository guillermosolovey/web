# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal academic website for Guillermo Solovey, built with **Quarto** and **R**. Supports two languages (English/Spanish) via Quarto profiles. Dynamic content (publications, teaching, group members, media) is fetched from Google Sheets at render time.

## Build Commands

```bash
# IMPORTANT: always render EN first, then ES.
# EN render wipes _site/ entirely (including _site/es/), so ES must go second.

quarto render --profile en
quarto render --profile es

# Full path on this machine (quarto not in PATH):
"C:\Program Files\RStudio\resources\app\bin\quarto\bin\quarto.exe" render --profile en
"C:\Program Files\RStudio\resources\app\bin\quarto\bin\quarto.exe" render --profile es
```

**Important:** Pause Dropbox before rendering. Dropbox locks files during sync and causes Quarto to fail when cleaning up temp files.

## Deploy workflow

1. Pause Dropbox
2. `quarto render --profile en` then `quarto render --profile es`
3. `git add _site/ _freeze/` and commit — **always include `_site/` after rendering or Netlify will deploy stale HTML**
4. `git push` → Netlify auto-deploys from the branch

**Branches:**
- `main` → live site at gsolovey.netlify.app
- `nueva-version` → staging branch at nueva-version--gsolovey.netlify.app

To publish: merge `nueva-version` into `main` and push.

## Architecture

### Multi-language structure

Two Quarto profiles produce parallel output:
- `_quarto-en.yml` → renders to `_site/` (English)
- `_quarto-es.yml` → renders to `_site/es/` (Spanish)

Both YAML files include the Google Fonts `<link>` for EB Garamond via `include-in-header`.

**Nav link rules:** All navbar hrefs must point to explicit `.html` files (e.g. `./es/index.html`, not `./es/`), otherwise Chrome shows a directory listing when browsing locally via `file://`.

Each `.qmd` file contains both languages using `content-visible` divs:
```markdown
::: {.content-visible when-profile="en"}
English text
:::
::: {.content-visible when-profile="es"}
Spanish text
:::
```

### Google Sheets data

All dynamic content is fetched with `gsheet::gsheet2tbl()`. Single spreadsheet:
`https://docs.google.com/spreadsheets/d/1id1P5ke9ZJckr_6ePP84FOLeQPoHb57eitX7yaZ_l8A`

| Sheet name    | gid        | Used in               | Key columns |
|---------------|------------|-----------------------|-------------|
| publicaciones | 1000751275 | research.qmd, media.qmd | type, title, author, year, journaltitle, abstract, url, pdf, github, osf |
| docencia      | 852428719  | teaching.qmd          | year, materia, url, carrera, cargo |
| grupo         | 1834640090 | grupo.qmd             | nombre, proyecto, carrera, fecha (actual/pasado) |
| media         | 625135132  | media.qmd             | year, title, medio, url, url2 |

**Filtering logic:**
- `research.qmd`: `type == "pre-print"` for preprints, `type == "Article"` for publications
- `media.qmd` outreach section: `type == "media"` from publicaciones sheet
- `teaching.qmd`: `cargo %in% c("prof", "doc")`
- `grupo.qmd`: `fecha == "actual"` and `fecha == "pasado"`

### Pages

| File         | EN navbar | ES navbar    |
|--------------|-----------|--------------|
| index.qmd    | home icon | home icon    |
| research.qmd | research  | investigación|
| teaching.qmd | teaching  | docencia     |
| grupo.qmd    | group     | grupo        |
| media.qmd    | media     | medios       |

CV also appears in both navbars: links to `CV-Solovey-eng.pdf` (EN) and `es/CV-Solovey.pdf` (ES).

### Styling

- Theme: `flatly` (Bootstrap) + `custom.scss`
- Font: **EB Garamond** (Google Fonts) for all headings; body stays in system sans
- Background: warm off-white `#faf9f7` (page, navbar, footer)
- Text: dark gray `#2d2d2d`
- Accent color: verde pizarra `#2a6e5a` (links, pill badges, self-author name, member carrera)

**Key CSS classes in `custom.scss`:**
- `.research-list` — wrapper div that activates the compact two-column grid layout (year left, content right). Used in research.qmd, teaching.qmd, media.qmd.
- `.pub-entry` — individual entry row. Inside `.research-list` renders as a grid.
- `.pub-cont` — added to entries that continue a year group (year_label == ""). Draws a left border on `.pub-body` to show grouping.
- `.pub-year-col` — year label in left column (shown only for first entry per year)
- `.pub-title-row`, `.pub-journal-row`, `.pub-author-row` — content rows within `.pub-body`
- `.pub-links a` — pill-style badges (pdf, github, osf, abstract toggle)
- `.abstract-body` — collapsible abstract block (Bootstrap collapse)
- `.self-author` — highlights "G. Solovey" in author lists (color + bold)
- `.members-grid` / `.member-card` — CSS grid for group members page
- `.apps-grid` / `.app-card` — CSS grid for interactive apps in teaching page

### Compact list format (research, teaching, media)

All list pages use the same pattern: raw HTML output from R via `cat()` with `results='asis'`, wrapped in a `::: {.research-list}` Quarto div (or `<div class="research-list">` directly in R output).

Year label appears only on the **first entry of each year group**; subsequent entries in the same year get class `pub-cont` which draws a left grouping border.

### research.qmd — render_entry()

Defined in the `load_packages` chunk and shared across preprints and pubs chunks. Takes:
- `p` — one row of the dataframe
- `use_pdf_col` — TRUE uses `p$pdf` for the pdf badge (local file), FALSE uses `p$url`
- `row_id` — unique string for Bootstrap collapse IDs
- `year_label` — year string or `""` (determines `pub-cont` class)

Entry structure: title (linked) → journal (own row, italic green) → authors → pill badges → collapsible abstract.

`G. Solovey` is replaced in the `author` column via `str_replace_all` with `<strong class="self-author">` before calling render_entry.

### teaching.qmd — interactive apps

The apps section is hardcoded HTML (not from a sheet) as `.apps-grid` / `.app-card` divs, duplicated inside `content-visible` blocks for EN and ES. Each card has `.app-name` and `.app-type` (experiment / visualization / game).

### grupo.qmd — member cards

Members rendered as `.members-grid` of `.member-card` divs. Current and past members use identical card style; section headers (h3) provide the distinction.

### Extensions and icons

Three Quarto extensions in `_extensions/`:
- `mcanouil/iconify` — general icons
- `quarto-ext/fontawesome` — FontAwesome icons (used in navbar)
- `schochastics/academicons` — academic platform icons (Google Scholar, ORCID, etc.)

### R packages required

`fontawesome`, `gsheet`, `tidyverse`

### Publications PDFs

PDF files live in `publications/` with naming convention `YYYY_AuthorLastname.pdf`.

## Pending improvements

- [ ] **Google Sheet privado** — migrate from `gsheet` to `googlesheets4` with service account auth
- [ ] **Auto-update via GitHub Actions** — scheduled workflow to render and deploy automatically
- [x] **Abstracts colapsables** — columna `abstract` en sheet, toggle Bootstrap collapse en research.qmd
- [ ] **Actividades de extensión** — agregar sección o página con contenido de la hoja gid=117290007 del Google Sheet
- [ ] **Figuras por paper** — agregar columna `fig` en el Google Sheet (publicaciones) con URL de imagen; implementar thumbnail a la izquierda de cada entrada en research.qmd. En espera de imágenes.
- [ ] **Charlas y presentaciones futuras** — agregar `type == "talk"` en el Google Sheet; mostrar en research.qmd con etiqueta "Upcoming". En espera de que Guillermo cargue datos.
- [ ] **Featured papers** — agregar columna `featured` (TRUE/FALSE) en el Google Sheet; mostrar sección destacada al tope de research.qmd con más peso visual. Infraestructura pendiente.
- [ ] **Unificar cuentas de GitHub** — `guillermosolovey` (personal) y `gsolovey-utdt` (Di Tella). Evaluar consolidar.
- [ ] **Merge a main** — cuando nueva-version esté lista, mergear a main para publicar en gsolovey.netlify.app

## Known issues

- Dropbox locks files during render — always pause Dropbox before running `quarto render`
- EN render wipes `_site/` — always render EN before ES
- `quarto` is not in PATH on this machine — use full path or run from RStudio terminal
- Git remote uses HTTPS (not SSH) — Windows Credential Manager handles auth
