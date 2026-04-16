# Architecture

The app is a static, zero-dependency single-page application. All markup, styles, and behavior live in [index.html](../index.html). There is no build step, no framework, and no runtime package dependency — a browser and a static file server are enough.

## Data flow

```
DrawTabData (submodule)
  └── data/inventory/sevenpens-{tablets,pens}.json
        │
        │ fetch() on DOMContentLoaded
        ▼
  remapTablets() / remapPens()   ← normalize field names
        ▼
  inventoryData = { Tablets: [...], Pens: [...] }
        ▼
  applySort()  ← filter → search → sort → render
        ▼
  renderTable()  → <tbody> rows
```

1. On `DOMContentLoaded`, the page fetches both JSON files in parallel via `Promise.all`.
2. Each raw record is run through a remap function that projects the source schema onto the field names this UI uses (e.g. `ModelId` → `ModelID`, `PenEntityId` → the last dotted segment as `Pen`).
3. The remapped arrays are stored in a single module-scoped `inventoryData` object.
4. The default tab (`Tablets`) is activated, which triggers filter population and a full render.

## Modules (within the single script)

- **Loading & remapping** — `remapTablets`, `remapPens`, and the `DOMContentLoaded` handler.
- **Filter population** — `populateBrandFilter`, `populateTypeFilter`. Rebuilt per tab; brand selection is preserved across tab switches when still valid, type selection always resets.
- **Search** — `parseSearchQuery` tokenizes the input (quoted phrases or whitespace-separated terms), then converts each token into a predicate (`includes` for plain terms, an anchored regex for terms containing `*` or `?`). `matchesSearch` ANDs predicates across a field list.
- **Sort** — `sortData` implements three comparators (`brand`, `date`, `modelId`), each with two secondary keys for stable ordering, and honors the direction dropdown.
- **Rendering** — `renderTable` clears the tbody and builds rows from a column list; empty results render a centered "No items found." placeholder.
- **Tab switching** — `openTab` toggles `.active` on the buttons, shows the matching `.tab-content`, updates `currentTab`, refreshes filters for the new tab, and calls `applySort`.

## State

All state is plain JavaScript variables at script scope:

- `inventoryData` — the remapped `{ Tablets, Pens }` snapshot, loaded once.
- `currentTab` — `'Tablets'` or `'Pens'`; drives which dataset the row count reads from.
- Everything else (search query, sort key, direction, filter selections) lives in the DOM and is read on demand in `applySort`.

Events from the search box, filters, and sort controls all fan into `applySort`, which is the single re-render entry point.

## Styling

All CSS is in a `<style>` block at the top of `index.html`. There are no external stylesheets, fonts, or icon sets — the close/clear button uses an inline `&#x2715;` glyph.

## Extending the app

- **New column** — add the field to the remap function, the `<th>` list in the table's `<thead>`, and the column array passed to `renderTable` in `applySort`.
- **New search field** — append it to `tabletSearchFields` or `penSearchFields` inside `applySort`.
- **New sort key** — add an `<option>` to `#sortOrder` and a branch in `sortData`.
- **New tab** — add a button and `.tab-content` div, extend `inventoryData`, and handle the new tab in `populateBrandFilter`, `populateTypeFilter`, `applySort`, and `openTab`.

## Non-goals

- No persistence of user UI state (selections reset on reload).
- No write path — the app is read-only over the JSON files.
- No pagination; the current dataset fits comfortably in a single render pass.
