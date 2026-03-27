# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal academic website for Guillermo Solovey, built with **Quarto** and **R**. Supports two languages (English/Spanish) via Quarto profiles. Dynamic content (publications, teaching, group members) is fetched from Google Sheets at render time.

## Build Commands

```bash
# IMPORTANT: always render EN first. The EN render wipes _site/ entirely (including _site/es/).
# Render English version first (default profile)
quarto render --profile en

# Then render Spanish version (outputs to _site/es/)
quarto render --profile es

# Preview with live reload
quarto preview
quarto preview --profile es
```

The `freeze: auto` setting means R code chunks are only re-executed when their source changes.

## Architecture

### Multi-language structure

Two Quarto profiles produce parallel output:
- `_quarto-en.yml` → renders to `_site/` (English)
- `_quarto-es.yml` → renders to `_site/es/` (Spanish)

**Nav link rules:** Language-switch links in the navbar must point to explicit `.html` files (e.g. `./es/index.html`, not `./es/`), otherwise Chrome shows a directory listing when browsing locally via `file://`.

The master `_quarto.yml` sets `profile.default: en` and groups both profiles together.

### Content files

Each `.qmd` file contains both English and Spanish content, separated using Quarto's `content-visible` divs:
```markdown
::: {.content-visible when-profile="en"}
English text
:::
::: {.content-visible when-profile="es"}
Spanish text
:::
```

### Dynamic data from Google Sheets

`research.qmd`, `teaching.qmd`, and `grupo.qmd` fetch live data from Google Sheets using the `gsheet` R package. Tables are rendered with `gt` and `gtExtras`. This means rendering requires internet access and R packages: `fontawesome`, `gsheet`, `gt`, `gtExtras`, `tidyverse`.

### Extensions and icons

Three Quarto extensions in `_extensions/`:
- `mcanouil/iconify` — general icons
- `quarto-ext/fontawesome` — FontAwesome icons
- `schochastics/academicons` — academic platform icons (Google Scholar, ORCID, etc.)

### Styling

`custom.scss` handles font sizing (h1–h3: 1.2–1.6rem), heading text shadows, and body colors. The Cosmo Bootstrap theme is the base.

### Publications

PDF files live in `publications/` and are referenced directly from `research.qmd`. New papers should be added there with naming convention `YYYY_AuthorLastname.pdf`.

## Deployment

The site is deployed by pushing `_site/` content to GitHub (`git@github.com:guillermosolovey/web.git`). There is no CI/CD — builds run locally before committing.
