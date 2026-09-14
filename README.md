# Gujarat District Summary — web dashboard

A browser-based version of the Summary sheet: same filters, same revenue table,
same segment compliance table and diagnostics — but genuinely dynamic (real
multi-select, no frozen-pane or dropdown quirks) and with the actual
interactive map embedded, not a static image.

## How it's built

- `index.html` — the dashboard page (filters, tables, layout, styling)
- `dashboard_logic.js` — the calculation engine (filtering, aggregation).
  Kept separate on purpose so it can be tested independently of the browser.
- `Gujarat_Summary_Data.xlsx` — the data source. **Edit this file directly**
  (add rows to the Detailed Sheet, update figures) and refresh the browser —
  no re-export, no rebuild step.
- `Gujarat_Hotspot_Map.html` — the original interactive map, embedded inline.

The dashboard reads the **Detailed Sheet** tab of the xlsx file each time it
loads, using a library called SheetJS (loaded automatically from the internet
the first time you open the page — after that your browser caches it).

## Running it — one thing you must do first

Opening `index.html` by double-clicking it will **not work**. Browsers block
web pages from reading local files directly for security reasons (this is a
browser rule, not something this dashboard can work around). You need to
serve the folder over a tiny local web server — this takes one command and
no installation if you already have Python or Node:

**Python (most common):**
```
cd path/to/this/folder
python3 -m http.server 8000
```
Then open **http://localhost:8000** in your browser.

**Node.js, if you have it instead:**
```
cd path/to/this/folder
npx serve .
```
Then open the URL it prints (usually http://localhost:3000).

**VS Code users:** the "Live Server" extension does this with one click —
right-click `index.html` → "Open with Live Server".

Leave the terminal window open while you use the dashboard; closing it stops
the server. To stop it yourself, press Ctrl+C in that terminal.

## Updating the data

1. Open `Gujarat_Summary_Data.xlsx` in Excel, edit the **Detailed Sheet** tab
   (add rows, correct figures, whatever's needed).
2. Save the file, keeping the same name and location.
3. Refresh the dashboard in your browser (F5). That's it — no export, no
   conversion, no re-running anything.

If you rename columns on the Detailed Sheet, update the matching field names
near the top of `dashboard_logic.js` (each one is named after its column
header, e.g. `r['BA Segment']`).

## What's live vs. what's a fixed reference

- Filters, revenue table, segment compliance table, and the Focus/KOP
  diagnostics all recompute instantly from whatever is in the xlsx file.
- The Sales Driver text for each segment (e.g. "Focus account coverage,
  scheme point achievement...") is fixed in `dashboard_logic.js`
  (`SEGMENT_DRIVERS`), matching how the original Summary sheet treated it as
  a static reference table, not something that changes with the data.
- The map is the same static hotspot classification built earlier in this
  project (from Housing.com listing data) — it does not update from the
  Detailed Sheet, since it's a different underlying dataset (locality listing
  prices, not your sales figures). It needs an internet connection to draw
  its street map tiles; if you're offline, the map area will appear blank
  until you're back online, but the rest of the dashboard works either way.

## What this fixes vs. the Excel version

- Locality selection is a real multi-select checkbox list, not a Yes/No grid
  workaround.
- No frozen-pane or dropdown-arrow quirks — those were Excel-specific
  rendering issues this format doesn't have.
- The map is the actual interactive version (pan, zoom, layer toggles), not
  a static image.

## Known limitation

Only the **Detailed Sheet** tab is read. The RSM reconciliation sheets, the
KOP-Q2 sheet, and the Focus Accounts sheet are not re-joined live — their
results are already folded into Detailed Sheet's `KOP:`, `Focus:`, and `RSM:`
columns from earlier work on this project, so editing those columns directly
is the way to update that information here.
