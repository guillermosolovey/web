# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal academic website for Guillermo Solovey, built with **Quarto** and **R**. Supports two languages (English/Spanish) via Quarto profiles. Dynamic content (publications, teaching, group members, media) is fetched from Google Sheets at render time.

## Build Commands

```bash
# IMPORTANT: always render EN first, then ES.
# EN render wipes _site/ entirely (including _site/es/), so ES must go second.

# Render English version first
quarto render --profile en

# Then render Spanish version (outputs to _site/es/)
quarto render --profile es

# Full path on this machine (quarto not in PATH):
"C:\Program Files\RStudio\resources\app\bin\quarto\bin\quarto.exe" render --profile en
"C:\Program Files\RStudio\resources\app\bin\quarto\bin\quarto.exe" render --profile es
```

**Important:** Pause Dropbox before rendering. Dropbox locks files during sync and causes Quarto to fail when cleaning up temp files.

The `freeze: auto` setting means R code chunks are only re-executed when their source changes.

## Deploy workflow

1. Pause Dropbox
2. `quarto render --profile en` then `quarto render --profile es`
3. `git add _site/ _freeze/` and commit
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
| publicaciones | 1000751275 | research.qmd, media.qmd | type, title, author, year, journaltitle, url, pdf, github, osf |
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

### Styling

- Theme: `flatly` (Bootstrap) + `custom.scss`
- `custom.scss`: white background, dark gray text (#2d2d2d), blue links (#2c5f8a)
- Entry format: `.pub-entry` divs with `.pub-links` pill-style badges for pdf/github/osf/links
- Year headers in research: `h4.anchored` at 1.25rem, color #444

### Publications format (research.qmd)

Uses a `render_entry()` helper. Publications grouped by year with `#### YEAR` headers rendered via `cat()` with `results='asis'`. Same pattern used in media.qmd, teaching.qmd, grupo.qmd.

### Extensions and icons

Three Quarto extensions in `_extensions/`:
- `mcanouil/iconify` — general icons
- `quarto-ext/fontawesome` — FontAwesome icons (used in navbar)
- `schochastics/academicons` — academic platform icons (Google Scholar, ORCID, etc.)

### R packages required

`fontawesome`, `gsheet`, `tidyverse`
(`gt` and `gtExtras` still loaded in some files but no longer used — can be removed)

### Publications PDFs

PDF files live in `publications/` with naming convention `YYYY_AuthorLastname.pdf`.

## Pending improvements

- [ ] **Google Sheet privado** — migrate from `gsheet` to `googlesheets4` with service account auth
- [ ] **Auto-update via GitHub Actions** — scheduled workflow to render and deploy automatically
- [ ] **Abstracts colapsables** — add abstract column to publicaciones sheet, implement toggle in research.qmd
- [ ] Remove `gt` and `gtExtras` dependencies (no longer used after switching to list format)
- [ ] Add `.gitattributes` to fix LF/CRLF warnings on Windows

## Known issues

- Dropbox locks files during render — always pause Dropbox before running `quarto render`
- EN render wipes `_site/` — always render EN before ES
- `quarto` is not in PATH on this machine — use full path or run from RStudio terminal
- Git remote uses HTTPS (not SSH) — Windows Credential Manager handles auth
