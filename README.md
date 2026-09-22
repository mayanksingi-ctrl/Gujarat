# Gujarat District Summary — web dashboard

Same architecture as the Pune dashboard, built for Gujarat's richer data —
Gujarat's Detailed Sheet has the Focus Accounts / KOP-Q2 / RSM Review /
leads extract all joined in, so this dashboard shows the full process-input
compliance table with real numbers, not N/A.

## What's inside

- `index.html` — the dashboard page
- `gujarat_logic.js` — the calculation engine (filtering, aggregation),
  kept separate so it can be tested independently of the browser
- `Gujarat_Summary_Data.xlsx` — the data source. **Edit this directly**
  (the "Detailed Sheet" tab) and refresh the browser — no rebuild step.
- `Gujarat_Hotspot_Map.html` — the interactive Gujarat hotspot map,
  embedded inline
- `gujarat_developers.json` — developer-by-locality lookup, extracted from
  the Developers tab in the xlsx

## Running it

Same rule as the Pune package: **double-clicking `index.html` will not
work.** Browsers block local file reads for security. Serve the folder with
a one-line local server:

**Python:**
```
cd path/to/this/folder
python3 -m http.server 8000
```
Then open **http://localhost:8000**.

**Node.js:**
```
npx serve .
```

**VS Code:** right-click `index.html` → "Open with Live Server".

## What's shown

- **Filters:** Zone, BA Type, BA Segment, Loyalty, Focus Account, KOP
  Account, Locality Category, Locality, Focus Coverage — the same nine
  filters as the Excel version. Locality is a single-select dropdown that
  narrows to whichever Locality Category is picked, matching the decision
  made earlier in this project to simplify away from a checkbox-based
  multi-select.
- **FY26-27 pro-rata comparison** — same mechanism as the Excel workbook.
  The 25-26 → 26-27 growth column compares 26-27's actual figure against a
  pro-rated slice of 25-26 (assuming even monthly spread) rather than the
  full year, since 26-27 is still a partial year. Edit the "data captured
  through" date to match your actual cutoff; recalculates live.
- **Revenue by District** — all 29 districts, with growth% columns shown as
  a genuine heat-map background gradient (red → amber → green).
- **Focus account dependence** and **KOP account dependence** — the same
  live diagnostic blocks as the Excel Summary sheet, year by year.
- **BA Segment process-input compliance** — Focus Accounts, Covered,
  Coverage %, Scheme Points Achieved/Target/%, and Leads, computed live
  from the real Focus/KOP-Q2/RSM/leads join in the Detailed Sheet.
- **Developers by locality** — always visible, shows what's been pulled so
  far (currently 2 of 688 localities: Vesu, Adajan) plus locality-specific
  detail when you pick one in the filter bar.
- **Map** — the existing interactive Gujarat hotspot map, needs an internet
  connection for its street tiles.

## Checking you're on the current build

The bottom-right of the page header shows a small "build" date/letter (e.g.
"build 2026-09-15-c"). If a fix described in this README doesn't seem to
be there, check that marker first — a hard refresh (Ctrl/Cmd+Shift+R) or
clearing the browser cache for localhost will usually resolve it, since
this is otherwise identical to a previous build with the same file names.

## Fixed since first published

**Made the narrowing impossible to miss.** The Locality Category → Locality
narrowing fix from before was correct and tested — but it wasn't very
visible, since the Locality dropdown still just displayed "(All)" after
narrowing, and you'd only see the shorter list by clicking it open. The
Locality label now shows a live count directly, e.g. "Locality (1 option
in category)", so it's obvious the moment you change the category, without
needing to open the dropdown.

**A serious one, not just a UI issue.** Gujarat's Detailed Sheet had never
had a "Locality Category" column materialized into it — every single row
was silently falling back to "Not classified", so the Locality Category
filter and the Focus/KOP compliance numbers tied to it were computed from
completely wrong groupings the whole time. Fixed by adding the real
column, computed from the Cube's verified Cluster → Category mapping (the
same one used throughout this project's Excel work). Re-verified against
the original figures: "Hotspot - High value" now correctly narrows to just
Sola, "Area of Interest" narrows to exactly 25 localities — both matching
the numbers established when this project first classified Gujarat's
hotspots.

Separately, the growth% heat map coloring had a bug: it scaled colors
relative to whichever min/max happened to be in the current filtered view,
so if every visible district had negative growth, the "least negative" one
could still render green. Fixed to anchor the color scale at a true 0%.

Also: the Locality dropdown didn't narrow when you picked a Locality
Category — it always showed the full list of ~690 localities regardless.
Fixed to rebuild its options against the selected category (and reset to
"(All)" if your previous pick isn't valid in the new category).

## Verified against established ground truth before publishing

Every figure below was independently re-derived from the raw Detailed
Sheet data and matched exactly against the numbers already established and
verified in the Excel version of this project:

- Grand Total: 1,041,399
- South Zone Grand Total: 478,375 (tested live via the Zone filter)
- R1 segment: 500 Focus Accounts, 13 Covered, 2,439 of 9,000 scheme points,
  19 leads
- Focus account dependence shares: 78.4% / 85.7% / 88.6% / 87.1%
  (23-24 through 26-27)
- KOP account dependence shares: 7.5% / 10.2% / 18.1% / 15.5%
- Pro-rata growth math cross-checked against the Excel workbook's own
  pro-rata block

## Focus Coverage refreshed (RSM_Review_Master, Sep-26 YTD)

Focus Account Coverage, and the "Covered" / "Coverage %" columns in the BA
Segment table, are now based on the RSM_Review_Master upload rather than
the original August snapshot — one month more recent (through September
2026), and matched by GSTIN instead of the fuzzier name-based approach the
original file needed (99.6% of its rows had a usable GSTIN).

Three real bugs were found and fixed while refreshing this, not just new
numbers layered on old logic:
- The Excel workbook's hidden Cube stored coverage as a bucket-level flag,
  which double- and triple-counted accounts sharing a bucket. Rebuilt to
  query a proper per-account detail sheet instead.
- A data-clearing bug of my own left stale sale-quantity values on accounts
  that should have shown as not-covered under the refreshed data — correct
  in Excel's own Yes/No column, but would have caused this dashboard to
  overcount, since it checks for a value's presence rather than reading
  that column directly.
- A segment-naming inconsistency ("Project", "Contractor", and "PMC" merge
  into "Project/Contractor" elsewhere in the workbook) wasn't being applied
  to the new coverage data, undercounting that segment specifically.

All 13 segments' Focus Account and Covered counts were cross-checked
between this dashboard's own JavaScript logic and the Excel workbook
independently — every one matches exactly (R1: 500 Focus / 5 Covered,
R2: 382/9, Office Furniture: 136/26, and so on). Grand Total remains
1,041,399, confirming nothing else was disturbed by this update.

## Focus Coverage filter removed, Segment table achievement columns added

Matching the Excel workbook: the Focus Coverage filter (which had a real bug — comparing a
number against text — causing every specific selection to show empty) has been removed
entirely. The Segment table now shows three new columns (Coverage/Points/Leads Param — whether
that input applies to the segment, per its Sales Driver) plus an Achieved column: Yes only if
every applicable parameter shows real activity (Coverage % > 0, Points % > 0, Leads > 0).

## Trend & Root Cause Analysis sections added (7 new cards)

Everything from the Excel workbook's "Trend & Root Cause Analysis" sheet, built as static
pre-computed tables (`gujarat_trend_data.json`) rather than live recomputation from raw rows,
since the underlying methodology (weighted locality index, RCA classification, upside
estimation) was built and validated in Python — mirroring it in JS risked subtle drift from the
Excel figures. Sections: (1) Zone-wise trend, (2) District-wise trend, (3) BA Segment trend,
(4) Root Cause Analysis / CAPA with Upside Potential (District x Segment, sorted by
cost-benefit), (4a) sample-size transparency for the upside estimates, (5) Retail
locality-based analysis using the new weighted Hotspot/Area of Interest classification,
(6) locality income segmentation (property-price proxy) with full locality detail, (7) kitchen
showroom competitive-presence data for the 16 High/Medium income localities.

All figures verified against the Excel workbook before publishing (e.g. Ahmedabad/R1 upside =
162,140 units, Hotspot-Medium growth = +12.1%, matching exactly).

## Map refreshed with new Hotspot classification, income view added

The map (`Gujarat_Hotspot_Map.html`) previously showed the *old* listing-only Hotspot
classification (Sola as High value, Adajan as Medium value) — it had not been touched since the
weighted reclassification was applied to the Excel workbook. It's now refreshed to match, and a
new "Income view" toggle has been added alongside the existing "Hotspot view" — same 28
localities, same map, but switching the button re-colors every marker by income band
(property-price proxy) instead of Hotspot category, with its own legend and description text.
Only 10 of these 28 localities have income data (from a separate, sales-volume-based research
set) — the rest show as neutral grey with "no income data researched" in the popup, rather than
guessing.

## Updating the data

Edit `Gujarat_Summary_Data.xlsx`'s "Detailed Sheet" tab directly (or the
"Developers" tab for the developer lookup), save, refresh the browser.
If you rename a column, update the matching field name near the top of
`gujarat_logic.js`.

## Known limitation

Only the "Detailed Sheet" and "Developers" tabs are read directly. The RSM
reconciliation sheets, KOP-Q2 sheet, and Focus Accounts sheet are not
re-joined live in the browser — their results are already folded into
Detailed Sheet's `KOP:`, `Focus:`, and `RSM:` columns from earlier work on
this project, so editing those columns directly is the way to update that
information here.
