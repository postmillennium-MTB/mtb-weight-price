# Weight vs. Price MTB Scatterplot — README

A single-file, dependency-free HTML/JS/CSS widget for Post Millennium Renaissance. Plots mountain bikes by price and weight across six categories (XC, Trail, Enduro, DH, Full Power eMTB, Lite eMTB), with an optional fit line, unit/currency toggles, a per-bike weight & price adjustment table, show/hide checkboxes, and a form to add your own bikes.

A seventh tab, **All six**, is a cross-category view: it puts every category's fit line on one pair of axes, labels each with whether its slope is statistically significant, and backs that up with a summary table (bikes, cost per lb shed, R², p-value, verdict, plain-English reading) and a "how to read this chart" section.

## How the data was built

All seeded bikes were researched via web search against manufacturer sites, Pinkbike, Vital MTB, and magazine reviews (MBR, MBA, EMTB Forums, BikeRadar, etc.) in July 2026. Nothing here is pulled from a live pricing API — it's a hand-assembled snapshot.

## Known limitations on accuracy

- **Prices are launch/list MSRP in USD**, mostly at the top-spec build level for each model. Actual street prices, sale pricing, regional pricing, and current-year price changes are not reflected. A bike shown at $10,999 may currently sell for less (the Ibis Oso is a clear example — it launched at $10,999 and has since dropped to $5,999–$7,999 depending on spec year).
- **Weights are manufacturer-claimed or single magazine-tested figures**, typically for a middle frame size (M/L) without pedals. Actual weight varies by frame size, wheel choice (29"/mullet), region-specific builds, and unit-to-unit variance. Treat each point as ±0.5–1 lb, not a precise figure.
- **Model years are approximate.** Bike brands don't use a consistent model-year convention — some spec sheets reflect a "2024" bike still sold into 2025, etc. Where a generation change was ambiguous, the most recent confirmed spec was used.
- **CHF conversion uses ECB reference rates, not live market rates.** The widget fetches from the Frankfurter API (free, keyless, CORS-enabled) on page load and caches the result in `localStorage` for 5 days. The ECB publishes once per working day around 16:00 CET, so the rate is end-of-day, current within a business day or so — never real-time. If the fetch fails, the widget falls back to a stale cached rate, then to a hardcoded constant (`CHF_PER_USD`, currently 0.80). The footer states which of the three is in use.
- **No live pricing/inventory feed.** Everything is hardcoded in the `DATA` object in the script. If a manufacturer discontinues a model or changes spec, the widget won't know.
- **Coverage is not exhaustive.** Each category includes roughly 13–20 bikes, weighted toward well-known, widely reviewed flagship builds. Many legitimate models (budget-tier builds, regional brands, alloy-frame versions of the same bike) are omitted.
- **The fit line is a simple linear regression** (least squares) on whatever bikes are currently checked "on." With only 13–20 points per category, it's sensitive to outliers — check the R² readout under the chart before treating the slope as meaningful, especially in categories with a wide price spread but few bikes at the extremes.
- **User-added bikes, weight/price adjustments, and checkbox states are session-only.** Nothing persists after a page refresh. If someone shares a link to the page, they get the default dataset, not your edits. (The FX rate is the one exception — it is cached.)

### The "All six" comparison tab and its significance test

The seventh tab draws all six fit lines together and labels each one significant or not. What that label does and doesn't mean:

- **The test is a two-tailed Student-t test on the regression slope** (null hypothesis: slope = 0), with n−2 degrees of freedom. It's computed in-file — `logGamma()`, `betaContinuedFraction()`, `incompleteBeta()` and `tTestPValue()` in the script, since the widget ships with no dependencies. The implementation was checked against published t-table critical values (t=2.228/df=10 → p=0.0500; t=2.086/df=20 → p=0.0500; t=3.169/df=10 → p=0.0100) before shipping. **If anyone rewrites those functions, re-check those numbers** — a subtly wrong p-value looks entirely plausible on screen and would silently mislabel a category.
- **"Not significant" means "this dataset can't tell," not "no relationship."** With 13–20 bikes per category the test has little power; a real but modest slope can easily fail to clear p < 0.05.
- **Six tests at once inflate the false-positive rate.** At α = .05 across six categories there's roughly a 26% chance at least one "significant" result is noise. Categories that also clear the Bonferroni-corrected threshold (p < 0.0083) are marked † in the table. That threshold is derived from `Object.keys(DATA).length`, so adding a seventh category re-tightens it automatically.
- **Significance says nothing about causation.** These are observational, cross-brand, mostly top-spec bikes; expensive builds differ in layup, wheels, drivetrain and suspension all at once. A steep, significant line means price and weight travel together in this snapshot, not that spending more makes a bike lighter.
- **The cost-per-lb figure is shown in parentheses where the slope isn't significant**, because a near-flat slope produces arithmetically real but meaningless numbers (the alloy-only Trail filter, for example, yields ~$300,000/lb). Where the slope points *upward* — Full Power eMTB, where more money buys more motor and battery — the column reads `n/a` instead.
- **The lines are clipped to each category's own observed price range.** No line is extrapolated. This is why they start and stop in different places, and it's deliberate: an extrapolated DH line running down to XC money would depict something that isn't in the data.
- **R² and p-values are invariant to the unit and currency toggles** (rescaling an axis can't create or destroy a relationship); the cost-per-unit figures are not.
- **The material filter and per-bike checkboxes feed this view too**, so filtering to Alloy leaves several categories with too few bikes to fit a line. The view says so rather than drawing one.

### Frame material specifically

Every bike carries a `mat` tag (`"carbon"` or `"alloy"`), driving the Carbon/Alloy/Both filter and the "carbon adds $X, sheds Y lbs" readout. Caveats worth knowing before leaning on that number:

- **The tag describes the main frame construction only.** Several bikes marketed as carbon use an alloy rear triangle or alloy links; they are tagged `carbon` here. Where a model exists in both materials, the tag reflects the *specific build listed*, not the platform.
- **Alloy coverage is thin, because the dataset is flagship-tier.** Current counts: XC 14C/0Al, Trail 14C/2Al, Enduro 15C/1Al, DH 12C/3Al, Full Power 12C/5Al, Lite eMTB 11C/1Al. The comparison stat is suppressed unless a category has at least 2 of each, so it currently only appears in Trail, DH, and Full Power.
- **The comparison is not a like-for-like frame swap.** It averages different bikes from different brands at different spec levels, so it conflates the material with everything else that differs between those bikes. A carbon Megatower and an alloy Meta differ by far more than frame material. Treat the number as a loose directional signal, not a material cost. The honest version of this stat requires same-model carbon/alloy pairs (Ripmo vs. Ripmo AF), which the dataset doesn't yet have.

## Suggestions for future updates

- **Consider confidence bands on the comparison chart.** The fit lines are drawn as single lines; a shaded ±1 SE band would make the difference between the tight XC/Trail/Lite fits and the loose Enduro/DH/Full Power ones visible without reading the table.
- **Consider comparing slopes to each other**, not just each slope to zero. "Is the XC slope steeper than the Trail slope?" is a different test (an interaction term) than the six the tab currently runs, and it's the question the chart visually invites.

- **Refresh pricing at least once a season.** MTB MSRPs shift with model-year changes (usually announced July–October) and mid-year price drops are common on 2–3 year old platforms.
- **Add a "last updated" timestamp** visible on the page itself, so anyone looking at it (including Town Council or nonprofit-partner audiences, if this ever gets reused for advocacy) knows how fresh the data is.
- **Add a data-source column or footnote per bike** (e.g., "Pinkbike review, June 2026") so any given price/weight figure is traceable back to where it came from — useful if this table is ever cited externally.
- **Consider persistent storage** (e.g., browser local storage, or PMR's backend if one exists) so a rider's own bike additions and weight tweaks survive a refresh, rather than resetting every session.
- **Expand alloy coverage — highest-value next step.** The material filter and the carbon-vs-alloy stat are both built and working, but they're data-starved. Adding same-model carbon/alloy pairs (Ibis Ripmo vs. Ripmo AF, Commencal Meta carbon vs. alloy, Santa Cruz C vs. CC vs. alloy) would turn the comparison stat from a loose signal into a real like-for-like material premium — and would populate the alloy filter in XC, Enduro, and Lite eMTB, where it's currently near-empty.
- **Consider a `platform` field** alongside `mat`, so same-model pairs can be identified programmatically and the stat could compute a true within-model average rather than a cross-brand one.
- **Expand model coverage generally**, especially mid-tier builds — the dataset skews toward top-spec, which pushes prices higher than what a typical rider actually pays. A "build tier" toggle (top-spec vs. mid-spec vs. entry) could make the price axis more representative.
- **Cross-check weights against a second source per bike** where possible (magazine test + manufacturer spec) and note any material gap, since claimed weights tend to run 1–3 lb optimistic versus independently tested weights.
- **Revisit category boundaries periodically.** The line between Trail/Enduro and Full Power/Lite eMTB keeps shifting as travel numbers and motor systems evolve; a bike correctly categorized today may sit oddly in a year or two.

## File

- `index.html` — the widget itself (repo: `postmillennium-MTB/mtb-weight-price`). All data lives in the `DATA` object near the top of the `<script>` block. FX config (`CHF_PER_USD` fallback, `FX_CACHE_KEY`, `FX_MAX_AGE_DAYS`) sits just above it.
- The PMR site wrapper lives at `pmr-website/mtb-weight-price/index.html` and iframes this repo's GitHub Pages URL.

## External dependencies

The widget is otherwise fully self-contained vanilla HTML/JS/CSS, with one runtime network call: the Frankfurter FX endpoint (`api.frankfurter.dev`, falling back to `api.frankfurter.app`). It degrades gracefully — if that call fails or is blocked, everything still works and CHF just uses the cached or fallback rate.
