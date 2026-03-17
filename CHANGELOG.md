# Changelog

All notable changes to this project are documented here. Dates reflect the data vintage used for seed prices, not necessarily the commit date.

---

## [3.0] — March 2026

### Added
- **Prediction markets tab** (renamed from "Kalshi compare") — now integrates both Kalshi and Polymarket
- **Polymarket integration** with seed data and independent live-fetch button:
  - Per-meeting decision markets (monthly fed-decision events)
  - Cut count distribution 2026 (13-outcome market, $10M volume)
  - Terminal rate end-2026 (polymarket what-will-the-fed-rate-be-at-the-end-of-2026)
  - Hike risk 2026 (polymarket fed-rate-hike-in-2026, $190K volume)
  - Emergency cut before 2027 (Polymarket-unique, no Kalshi equivalent)
  - First-cut timing per meeting (polymarket fed-rate-cut-by-629, 8 outcomes)
  - Fed Chair nomination (polymarket who-will-trump-nominate-as-fed-chair, $230M volume)
- **Nine panels** total (up from six):
  - ① Meeting divergence: 3 bars per meeting (CME / Kalshi / Polymarket) with Kal−CME and Poly−Kal basis pills
  - ② Cut count: 3-bar histogram across 6 buckets (0–5+) with three-way expected cut comparison and all three basis spreads
  - ③ Combo path decomposer (unchanged, Kalshi)
  - ④ Market-implied dot plot (unchanged, CME + Kalshi + Fed SEP)
  - ⑤ Hike risk gauge: three concentric arcs (CME / Kalshi / Polymarket)
  - ⑥ First-cut timing: CME step curve + Kalshi year-end circles + Polymarket per-meeting diamonds
  - ⑦ Terminal rate distribution: CME tree bars vs Polymarket upper-bound bars (adjusted −0.25% for comparability)
  - ⑧ Fed Chair nomination: hawkishness-tagged bar chart (Kevin Warsh hawkish / Rick Rieder dovish)
  - ⑨ Emergency cut gauge: Polymarket-unique bar showing unscheduled FOMC action probability
- **Polymarket color** (cyan `--c-poly`) added to CSS root and dark-mode variants
- **Kalshi–Polymarket informational spread**: basis pills showing the cross-platform disagreement, structurally separating institutional/domestic (Kalshi) from global/retail (Polymarket) sentiment

### Changed
- Tab renamed from "⚖ Kalshi compare" to "⚖ Prediction markets"
- `getMeetingComps()` extended to return Polymarket P(cut) and both basis spreads
- `panel2Svg()` updated to accept three data sources
- `panel5Svg()` updated with third arc for Polymarket hike probability
- `panel6Svg()` updated with Polymarket cyan diamonds for per-meeting first-cut timing

## [2.0] — March 2026

### Added
- **SOFR strip tab** (5th tab): 3-month SOFR futures (SR3) analysis with three sub-panels:
  - Editable SR3 contract table (SR3M6–SR3Z7, Jun 2026–Dec 2027) with live ZQ quarterly avg EFFR comparison and basis in basis points
  - Three-curve forward rate chart: SR3 implied SOFR (green), ZQ quarterly avg EFFR (blue), instantaneous 3M forward rates (dashed green), Fed December 2025 SEP dots (purple)
  - SOFR–EFFR basis bars by quarter with strip summary statistics (near-term basis, terminal rates, strip slope, forward trough quarter)
- **SR3 live refresh**: dedicated Fetch SR3 button using Anthropic API + web search, independent of ZQ refresh
- **Instantaneous forward rate**: strips the 3-month averaging effect from adjacent SR3 contracts to expose the marginal rate the market prices for each successive quarter
- **Version badge** in header, updated on each data refresh
- **GitHub Pages support**: `index.html` redirect at repo root; enable Pages at `Settings → Pages → main / (root)` for live URL at `davidbieri.github.io/fed-funds`
- **Improved tab bar**: wraps on narrow screens

### Changed
- Subtitle updated to reflect 5 tabs
- `fwSwitch()` updated to handle `sofr` tab
- CSS: added `--c-sofr` (green) and `--c-sofr-fwd` (light green) color variables for SOFR-specific chart elements, with dark-mode variants

---

## [1.0] — March 2026

### Added
- Full **CME FedWatch methodology** implemented in-browser from first principles:
  - 12-month forward horizon (Apr 2026 – Mar 2027)
  - Corrected FOMC schedule with N/M day parameters
  - EFFR chain propagation via anchor non-meeting months
  - Binary probability tree with unconditional cumulative lattice
  - Summary statistics: P(≥1/2/3 cuts), expected cuts, implied rate
- **Four tabs**: Futures inputs, EFFR chain, Probability tree, Kalshi compare
- **Kalshi comparison tab** with six panels:
  1. Meeting-by-meeting divergence: CME vs Kalshi P(cut) with basis pills
  2. Cut count distribution: CME tree vs Kalshi `kxratecutcount-26dec31`
  3. Combo path decomposer: top 8 CME paths vs Kalshi `kxfedcombo` contracts
  4. Market-implied dot plot: CME chain vs Kalshi rates vs Fed SEP (3-curve)
  5. Hike risk monitor: semicircular gauge CME vs Kalshi `kxfedhike`
  6. First-cut timing: cumulative P(cut) step function vs Kalshi binary contracts
- **Live ZQ refresh**: Anthropic API + web search fetches current CME ZQ prices and FOMC meeting schedule
- **Live Kalshi refresh**: independent fetch for all Kalshi market data
- **Standalone operation**: API key input field for use outside claude.ai
- **Dark mode** via CSS `prefers-color-scheme`
- **README.md** with methodology documentation, usage instructions, data sources
