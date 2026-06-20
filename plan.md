# M365 Plans & Features Map

An auto-updating static site that maps Microsoft 365 products (top-level SKUs) to the service plans (features) they include. Sourced from Microsoft's official licensing CSV, hosted on GitHub Pages, refreshed weekly.

## Goal

Browse top-level products (Business Basic/Standard/Premium, E3/E5, F3, and the rest), drill into each to see its included features as tidied friendly names, and search across both products and features (including by GUID).

## Data source

Single CSV, published by Microsoft, confirmed fetchable:

```
https://download.microsoft.com/download/e/3/e/e3e9faf2-f28b-490a-9ada-c6089a1fc5b0/Product%20names%20and%20service%20plan%20identifiers%20for%20licensing.csv
```

### Shape (verified against the live file)

The CSV is already exploded: one row per (product, service plan) pair. Columns:

| Column | Meaning |
|---|---|
| `Product_Display_Name` | Top-level product, e.g. "Microsoft 365 Business Standard" |
| `String_Id` | SKU code, e.g. `O365_BUSINESS_PREMIUM` |
| `GUID` | Product SKU id |
| `Service_Plan_Name` | Feature code, e.g. `EXCHANGE_S_FOUNDATION` |
| `Service_Plan_Id` | Feature GUID |
| `Service_Plans_Included_Friendly_Names` | Feature friendly name, e.g. "Exchange Foundation" |

Numbers from the live file:
- 607 distinct products
- 5979 (product, plan) rows
- 515 products have 2 or more child plans, 92 have exactly 1

Grouping rows by `Product_Display_Name` reconstructs the hierarchy with no extra modelling. The friendly name is one per row already, so no in-cell newline parsing is needed (the per-row CSV is cleaner than the web table).

## Architecture

Three files. No build step, no framework, no runtime dependencies.

```
/index.html                      UI + CSV parse + render, all inline
/data.csv                        the MS file, committed verbatim by the Action
/.github/workflows/refresh.yml   weekly cron: curl the CSV, commit if changed
```

### How it updates

1. `refresh.yml` runs on a weekly cron. It curls the CSV to `data.csv` and commits only if the bytes changed.
2. That commit triggers the normal Pages redeploy. No content rebuild step exists, because the page reads the CSV in the browser.
3. Git history becomes a free changelog: each weekly diff shows exactly what Microsoft added or renamed.

Workflow needs `permissions: contents: write`. Pages is enabled once via repo settings (deploy from `main`).

### Why this shape

- Browser-side fetch direct from Microsoft would hit CORS and leave no version history. Committing the CSV fixes both.
- No JSON transform: the CSV is small and clean, parsed client-side on load.
- No CSV parser dependency: a short quoted-field-aware parser is enough. Add a library only if a real row breaks it.

## UI behaviour

### Layout
- A single scrollable list of products (the top level).
- Each product is a collapsible row. Expanding reveals its child features as a plain indented list of tidied friendly names.
- Search box at the top filters two ways:
  - typing a product name narrows the product list
  - typing a feature name shows only products containing it, auto-expanded
  - pasting a GUID (product or feature) resolves to the matching name

### Decisions captured from discussion
- **Scope:** show all products, default-filtered to the headline SKUs, with a "show everything" toggle. The default filter keys off `String_Id` naming patterns (not a hand-maintained list, so it does not rot). Toggle reveals the full 607.
- **Tidy feature names:** strip the trailing GUID from the friendly name for display, keep it searchable, reveal it on hover/click. Friendly names that read "RETIRED - ..." stay flagged as retired in text.

## Design constraints (from design.md)

- Palette and roles:
  - Primary accent (structure, links, default icons, active states): `#f06543` (tomato)
  - Secondary accent (supporting emphasis, same warm family): `#f09d51` (sandy brown)
  - Neutrals (surfaces, borders, dividers): `#e8e9eb` (platinum), `#e0dfd5` (soft linen)
  - Text (highest contrast): `#313638` (gunmetal)
- Hard corners, straight lines. No rounded-corner card grids.
- No status pills. Differentiate with colour from the one family, or with text, never an outside hue.
- No emoji. No small caps headings. No em-dashes in any text.
- Subtle UX, no on-hover animation noise, no explanation needed.
- WCAG 2: text contrast at least 4.5:1 (3:1 for large text). Gunmetal on platinum/linen passes; tomato is used for structure and accents, with contrast checked where it carries text.
- Icons: pull Font Awesome Free SVGs from the source repo and embed inline, recoloured to the palette. Likely needs: a chevron/caret for expand/collapse and a magnifying glass for search. Sources will be stated in the build summary.
- Data display must be self-evident. Child counts shown as labelled values, not as a chart.

## Non-trivial logic that gets a self-check

- The default SKU filter (which `String_Id` patterns count as "headline"). One assert-based check over a handful of known SKU codes so a pattern change that drops Business Standard fails loudly.
- The GUID-strip-but-keep-searchable transform. One check that a known "Name (guid)" string renders clean and still matches a GUID search.

## Open questions for review

1. Default SKU filter: confirm pattern-based is acceptable over a hand-curated list. Yes
2. Should retired plans be hidden by default, or shown with a "retired" text marker? retired colour marker
3. Any need to show the SKU `String_Id` / GUID on the product row itself, or only on demand? give the option

## Explicitly skipped (YAGNI)

- Build pipeline / pre-computed indexes: add when the CSV grows large enough that browser parse lags.
- CSV parser library: add when the inline parser fails on a real row.
- Search index library: 6000 rows filter fine in plain JS.
- Backend / database: none needed for a read-only published dataset.
