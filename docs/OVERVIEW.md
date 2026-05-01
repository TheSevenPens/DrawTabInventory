# Overview

> **⚠ Deprecated** — this project has been replaced by
> [DrawTabDataExplorer](https://thesevenpens.github.io/DrawTabDataExplorer/),
> which offers a strict superset of features. Use the
> [Tablet Inventory](https://thesevenpens.github.io/DrawTabDataExplorer/tablet-inventory)
> and
> [Pen Inventory](https://thesevenpens.github.io/DrawTabDataExplorer/pen-inventory)
> sub-tabs instead. This document is kept for historical reference.

DrawTabInventory is a single-page web app that displays a personal inventory of drawing tablets and stylus pens. It loads JSON data from the [DrawTabData](https://github.com/TheSevenPens/DrawTabData) repository (vendored as a git submodule at `data-repo/`) and renders it in two tabs: **Tablets** and **Pens**.

## Features

- **Two tabbed views** — separate tables for tablets and pens.
- **Search** — multi-term AND-matching with quoted phrases (`"pen display"`) and wildcards (`kam*`, `h?ion`).
- **Filters** — by brand and by type (tablet type or pen technology), scoped to the active tab.
- **Sort** — by brand, model ID, or order date, ascending or descending, with stable secondary keys.
- **Row count** — shows `Showing N of M items` for the current tab.

## Running locally

The page fetches JSON from `data-repo/data/inventory/`, so it must be served over HTTP (file:// triggers CORS errors). From the repo root:

```sh
python -m http.server
```

Then open <http://localhost:8000/>.

## Repository layout

- `index.html` — the entire app (HTML, CSS, JS in one file).
- `data-repo/` — git submodule pointing at `DrawTabData`; the inventory JSON lives under `data-repo/data/inventory/`.
- `.vscode/` — launch and task configs for local development.
- `docs/` — this documentation.

## Data source

- `data-repo/data/inventory/sevenpens-tablets.json` — object with an `InventoryTablets` array.
- `data-repo/data/inventory/sevenpens-pens.json` — object with an `InventoryPens` array.

Updating the inventory means updating the `DrawTabData` submodule, not editing files in this repo.

## Keeping the data up to date

The `data-repo/` submodule is pinned to a specific commit, so it does not update automatically when `DrawTabData` changes upstream. Pull periodically:

```sh
git submodule update --remote data-repo
git add data-repo && git commit -m "Bump DrawTabData submodule"
```

If you just cloned this repo and `data-repo/` is empty, initialize it first with `git submodule update --init`.

## Related project: DrawTabDataExplorer

[DrawTabDataExplorer](https://github.com/TheSevenPens/DrawTabDataExplorer) is a sibling project that often carries the **canonical** way of reading and interpreting the `DrawTabData` schema — it tends to stay more current with schema changes and covers more of the data surface than this inventory page does. When extending this app, adding a new field, or debugging unexpected JSON shapes, check DrawTabDataExplorer first as a reference or point of comparison.
