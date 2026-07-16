# Weight vs. Price MTB Scatterplot — README

A single-file, dependency-free HTML/JS/CSS widget for Post Millennium Renaissance. Plots mountain bikes by price and weight across six categories (XC, Trail, Enduro, DH, Full Power eMTB, Lite eMTB), with an optional fit line, unit/currency toggles, a per-bike weight & price adjustment table, show/hide checkboxes, and a form to add your own bikes.

## How the data was built

All seeded bikes were researched via web search against manufacturer sites, Pinkbike, Vital MTB, and magazine reviews (MBR, MBA, EMTB Forums, BikeRadar, etc.) in July 2026. Nothing here is pulled from a live pricing API — it's a hand-assembled snapshot.

## Known limitations on accuracy

- **Prices are launch/list MSRP in USD**, mostly at the top-spec build level for each model. Actual street prices, sale pricing, regional pricing, and current-year price changes are not reflected. A bike shown at $10,999 may currently sell for less (the Ibis Oso is a clear example — it launched at $10,999 and has since dropped to $5,999–$7,999 depending on spec year).
- **Weights are manufacturer-claimed or single magazine-tested figures**, typically for a middle frame size (M/L) without pedals. Actual weight varies by frame size, wheel choice (29"/mullet), region-specific builds, and unit-to-unit variance. Treat each point as ±0.5–1 lb, not a precise figure.
- **Model years are approximate.** Bike brands don't use a consistent model-year convention — some spec sheets reflect a "2024" bike still sold into 2025, etc. Where a generation change was ambiguous, the most recent confirmed spec was used.
- **CHF conversion is a fixed constant** (`CHF_PER_USD` in the script, currently 0.80), not a live exchange rate. It will drift from the real market rate over time and needs manual updating.
- **No live pricing/inventory feed.** Everything is hardcoded in the `DATA` object in the script. If a manufacturer discontinues a model or changes spec, the widget won't know.
- **Coverage is not exhaustive.** Each category includes roughly 13–20 bikes, weighted toward well-known, widely reviewed flagship builds. Many legitimate models (budget-tier builds, regional brands, alloy-frame versions of the same bike) are omitted.
- **The XC tab's Devinci entry (Django) is a downcountry bike, not a true XC race bike** — Devinci doesn't currently make one. It's included so every category has a Devinci model, but it will sit off the cluster of dedicated XC race bikes on price and weight. Worth removing if you want that tab to stay strictly apples-to-apples.
- **The DH tab's "Propain Rage"** was added as requested.
- **The fit line is a simple linear regression** (least squares) on whatever bikes are currently checked "on." With only 13–20 points per category, it's sensitive to outliers — check the R² readout under the chart before treating the slope as meaningful, especially in categories with a wide price spread but few bikes at the extremes.
- **User-added bikes and weight/price adjustments are session-only.** Nothing persists after a page refresh. If someone shares a link to the page, they get the default dataset, not your edits.

## Suggestions for future updates

- **Refresh pricing at least once a season.** MTB MSRPs shift with model-year changes (usually announced July–October) and mid-year price drops are common on 2–3 year old platforms.
- **Add a "last updated" timestamp** visible on the page itself, so anyone looking at it (including Town Council or nonprofit-partner audiences, if this ever gets reused for advocacy) knows how fresh the data is.
- **Move the exchange rate to a live source.** A small fetch to a free FX API (updated daily, cached) would keep CHF accurate without a manual edit — though that adds a dependency PMR's current architecture avoids, so weigh that tradeoff.
- **Add a data-source column or footnote per bike** (e.g., "Pinkbike review, June 2026") so any given price/weight figure is traceable back to where it came from — useful if this table is ever cited externally.
- **Consider persistent storage** (e.g., browser local storage, or PMR's backend if one exists) so a rider's own bike additions and weight tweaks survive a refresh, rather than resetting every session.
- **Expand model coverage**, especially alloy/mid-tier builds — right now the dataset skews toward top-spec carbon builds, which pushes prices higher than what a typical rider actually pays. A "build tier" toggle (top-spec vs. mid-spec vs. entry) could make the price axis more representative.
- **Cross-check weights against a second source per bike** where possible (magazine test + manufacturer spec) and note any material gap, since claimed weights tend to run 1–3 lb optimistic versus independently tested weights.
- **Revisit category boundaries periodically.** The line between Trail/Enduro and Full Power/Lite eMTB keeps shifting as travel numbers and motor systems evolve; a bike correctly categorized today may sit oddly in a year or two.

## File

- `bike_weight_price.html` — the widget itself. All data lives in the `DATA` object near the top of the `<script>` block; the CHF rate is the `CHF_PER_USD` constant just above it.
