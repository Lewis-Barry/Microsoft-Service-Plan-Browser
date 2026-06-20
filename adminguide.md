# Admin guide

How to run and maintain the M365 Plans and Features Map.

## What this is

A single static page (`index.html`) that reads `data.csv` (Microsoft's official
licensing service plan reference) in the browser and lets you browse products and
the service plans they include, in either direction.

Three files, no build step, no dependencies:

| File | Purpose |
|---|---|
| `index.html` | The whole app: CSV parse, grouping, search, render. All inline. |
| `data.csv` | Microsoft's licensing file, committed verbatim. |
| `.github/workflows/refresh.yml` | Weekly job that re-downloads the CSV and commits it if it changed. (Added at the GitHub Pages stage.) |

## Running it locally

The page fetches `data.csv`, so it must be served over http, not opened as a
`file://` path.

```
cd /path/to/repo
python3 -m http.server 8765
```

Then open <http://localhost:8765/index.html>.

## Refreshing the data by hand

The weekly Action does this for you (see `.github/workflows/refresh.yml`). To do
it manually, scrape the Learn page for whatever CSV link it currently points at,
rather than hardcoding the download path (the path changes over time):

```
PAGE="https://learn.microsoft.com/en-us/entra/identity/users/licensing-service-plan-reference"
URL=$(curl -sL "$PAGE" | grep -oiE 'https://[^"]*\.csv' | head -1)
curl -sL "$URL" -o data.csv
```

Commit the result. The git diff shows exactly what Microsoft changed.

## Maintaining the feature name aliases

Microsoft's friendly names sometimes lag the names used on sales pages. The
clearest example: the plan sold as **Defender for Endpoint P2** appears in the
data only as "Microsoft Defender for Endpoint" (its plan code is `WINDEFATP`).

To relabel a plan, edit the `ALIASES` map near the top of the `<script>` in
`index.html`. It is keyed by the plan's GUID (`Service_Plan_Id`):

```js
const ALIASES={
  "871d91ec-ec1a-452b-a83f-bd76c7d770ef":"Microsoft Defender for Endpoint (Plan 2)", // WINDEFATP
  // add more here, one per line
};
```

The override applies everywhere: both views and search use the new name.

### Finding the GUID for a plan you want to rename

Search the page (with "Show SKU codes and GUIDs" turned on) for the plan, and
copy the GUID shown on its row. Or grep the CSV:

```
grep -i "defender for endpoint" data.csv
```

The third comma-separated field on the matching row is the product GUID; the
`Service_Plan_Id` (the GUID you want for the alias) is the fifth field. Confirm
you have the right one by checking the plan code in the fourth field
(`Service_Plan_Name`).

### Keep the alias map small

Only add aliases for plans whose marketing name genuinely differs and matters to
your users. Most names in the data are already current (Entra ID P1/P2, Defender
for Cloud Apps, Defender for Office 365 Plan 1/2). This is hand-maintained data,
so every entry is a small ongoing cost.

## The default product filter

The landing view shows only "headline" SKUs. This is a heuristic in `index.html`
(the `INCLUDE` and `EXCLUDE` regexes), not a hand-curated list, so it does not
need updating when Microsoft adds a SKU. If a product you care about is wrongly
hidden or a noisy one is wrongly shown, adjust those two regexes. The "Show all
products" toggle always bypasses the filter and shows all 600+ products.

## How the data is grouped

- **By product:** one row per `Product_Display_Name`, its plans listed beneath.
- **By feature:** one row per friendly name. Microsoft reuses a single friendly
  name across several plan GUIDs and in different letter casing; these are merged
  into one row, and the distinct GUIDs are shown when SKU codes are turned on. A
  feature is marked retired only if every GUID under that name is retired.
