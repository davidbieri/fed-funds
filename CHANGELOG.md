# Changelog

All notable changes to this project are documented here. Dates reflect the data vintage used for seed prices, not necessarily the commit date.

---

## [Unreleased]

### Changed — Landing Page Fixes (Session INDEX-J)
- Session INDEX-J: hero title em light-mode contrast fix (dark opacity token);
  all 11 overview mini-cards now have identity-colored left borders + --*-text
  title tokens (cards 01-06 were missing borders); Tab 10 renamed to
  "Balance Sheet: QE · QT Monitor" across index.html, README.md,
  CLAUDE.md, CHANGELOG.md

### Changed — Landing Page CSS (Session INDEX-A)
- CSS theme system: light/dark toggle via `data-theme` attribute on `<html>`
- Sticky nav with backdrop blur, `.active` link state, `.scrolled` shadow, `.nav-theme` toggle button
- Tab card identity color system (`--card-accent` per card, `.tab-num` inherits accent color)
- Typography scale +1–2px across all elements (section-label, tab-group-head, hero-sub, stat-n, etc.)
- `.tab-new` badge class for recently added tabs
- `.mkt.mpt` + `.mkt.fred2` source card classes (Taylor rose, Financial teal)
- Theme-aware modal backdrop (`--modal-bg`, `--modal-backdrop`)
- Scenario table: `var(--surface2)` for hold pips and hover rows, `var(--border)` for cell borders

### Changed — Landing Page HTML (Session INDEX-B)
- `data-theme="light"` default on `<html>`, sticky nav with 4 anchor links (#tabs #sources #guide #cite) + theme toggle button
- Hero stat 10→11 tabs, hero eyebrow updated with Balance Sheet + Financial Stability
- Section `id` attributes added for scroll-linked nav highlighting
- All 11 tab cards assigned `tc-*` identity classes, Tab 10 inline style removed
- Tab 10 (Balance Sheet) and Tab 11 (Financial Stability) `.tab-new` badges
- Atlanta Fed MPT → `.mkt.mpt`, FRED → `.mkt.fred2` (inline styles removed)
- Section label: "Eleven analytical tabs"
- IntersectionObserver scroll-highlighting, nav scroll shadow, theme persistence via localStorage
- All `v4.0` strings bumped to `v4.1` (nav badge, modal bar, citations, footer)

---

## [4.1] — March 2026

### Added — Balance Sheet: QE · QT Monitor (Tab 10)

- **BalanceSheetTab**: 4-panel layout with Fetch H.4.1 button (8 FRED series)
- Panel 1: Composition stacked bars (current vs neutral floor), KVJ (2011) 4-channel shadow rate decomposition (duration, safety, signaling, MBS), editable neutral floor + monthly runoff
- Panel 2: Monthly runoff pace vs cap (Treasury + MBS) with cumulative MBS undershoot
- Panel 3: Corridor mechanics — IORB, EFFR, Stein (2012) SVR, DSS (2018) liquidity premium, RRP with 3-tier regime coloring
- Panel 4: Two-instrument policy space scatter — 9 historical episodes (Pre-GFC through 2023), current "Now" position, neutral floor reference, combined stance (EFFR + KVJ shadow)

### Added — Financial Stability (Tab 11)

- **FinStabTab**: 7-panel layout with NFCI regime band and Fetch FRED button (3 series)
- Panel 1: NFCI horizontal gauge with 3 zones (loose/neutral/tight), stat cards (NFCI, HY OAS, xccy basis, editable lambda_fc)
- Panel 2: Finance-neutral Taylor Rule gap — 5 rate dots (EFFR, JR, fin-neutral, ZQ terminal, SEP dot), leaning premium shaded region
- Panel 2b: Gap decomposition stacked bar (JR base gap + Borio r* correction), leaning premium arrow, editable credit-GDP gap and productivity drag
- Panel 3: McCauley (2019) dollar safe asset gauge — 3-layer bars (reserves/GDP, swap line drawdowns, EUR/USD xccy basis)
- Panel 4: Doom loop scatter (term premium vs HY OAS, DSS LP bubble size, SS put threshold)
- Panel 4b: Policy feedback circuit — 6-node directed graph (EFFR/LP/HY/NFCI/FN/TP)
- Panel 5: Financial Cycle Phase Clock — 6-axis hexagonal radar, shoelace area index (0-100)

### Added — Math layer (10 pure functions)

- `qtShadowAdjChannels()`: KVJ (2011) 4-channel shadow rate decomposition
- `steinSVR()`: Stein (2012) shadow value of reserves (EFFR - IORB)
- `dssRiskChannel()`: DSS (2018) liquidity premium, implied leverage, risk premium effect
- `nsSafetyPremium()`: KVJ (2012) Treasury supply safety premium
- `finNeutralOutputGap()`: BDJ (2013) finance-neutral output gap
- `finCycleAdjustedRstar()`: Borio (2016 Pillar II) productivity-drag r* correction
- `finNeutralRate()`: Borio (2012/2014) JR ECM + lambda*NFCI augmentation
- `leaningPremium()`: finance-neutral vs market-implied terminal gap
- `gradualismGap()`: Stein & Sunderam (2018) SEP dot vs ZQ terminal
- `ssTermPremiumPutSignal()`: SS (2018) term premium put signal (+1/0/-1)
- `qtYieldShock()`: KVJ (2011) + D'Amico-King (2013) QT yield curve loadings

### Added — Cross-tab integration

- **Tab 07 (Yield Curve)**: NS beta0 decomposition (r* + pi* + psi_safety + psi_leverage + residual); QT shock curve overlay (sky-blue dashed) with shock impact table row
- **Tab 08 (Taylor Rule)**: 6th variant (finance-neutral, teal dashed); Borio r* toggle checkbox; finance-neutral column in decomposition table and rule comparison matrix
- **Tab 06 (Dashboard)**: 5 new regime rows (Stein SVR, DSS LP, combined stance, SS gradualism gap, Borio leaning premium)
- **Tab 09 (Riding)**: 6th filter (SS term-premium put signal)

### Added — Data layer

- `bsD` state object (14 fields): Fed H.4.1 weekly balance sheet data
- `fsD` state object (19 fields): financial stability indicators + model parameters
- `fetchBalanceSheet()`: 8 FRED series (WALCL, WTREGEN, WMBSEC, RRPONTSYD, WRESBAL, DFEDTARL, DFF, DTB3)
- `fetchFinancialStability()`: 3 FRED series (NFCI, SWPT, BAMLH0A0HYM2)
- 4 new reducer actions: APPLY_BALANCE_SHEET, APPLY_FIN_STAB, SET_BSD, SET_FSD
- 9 Phase 6 CSS tokens with dark-mode variants; tab nav divider between analytics groups

### Academic references (Phase 6)

- Afonso, Giannone, LaRocca & Packer (2022) — reserve demand / neutral floor estimation
- Borio (2012, 2014, 2016 Cato Journal) — finance-neutral r*, financial cycle, leaning against the wind
- Borio, Disyatat & Juselius (2013, BIS WP404; 2017 Oxford Econ Papers) — finance-neutral output gap
- D'Amico & King (2013) — local supply effects on yield curve (flow/stock QE channel)
- Drechsler, Savov & Schnabl (2018, JoF) — monetary policy and risk premia via liquidity premium
- Krishnamurthy & Vissing-Jorgensen (2011, BPEA; 2012, JPE) — QE channels (duration, safety, signaling, MBS)
- McCauley (2019, BIS WP 782) — dollar as global safe asset, 3-layer stress architecture
- Mehrling (2016) — BIS financial cycle framework, dollar doom loop
- Nagel (2016, QJE) — near-money convenience yield as liquidity premium proxy
- Stein (2012, QJE) — shadow value of reserves, two-instrument monetary policy
- Stein & Sunderam (2018, JoF) — Fed gradualism, term premium put, bond market volatility aversion

---

## [4.0] — March 2026

### Added — Policy Confidence Index (PCI)

- Four-component synthetic index (0-100): signal strength (ZQ strip slope),
  transmission quality (SOFR-EFFR basis), options conviction (Atlanta Fed MPT IQR),
  sentiment alignment (crowd vs ZQ gap)
- Regime labels: Uncertain / Divergent / Forming / Convicted
- Dashboard PCI stat row (3 cards) + Policy Confidence Dial SVG
- Dashboard sentiment gap sparklines (per-meeting crowd vs ZQ)
- Prediction Markets panel 10: PCI breakdown bars, platform divergence tracker,
  conviction spread gauge
- fetchMPT(): Atlanta Fed MPT distribution data via Anthropic API + web_search
- fetchFRED(): NFCI, SWPT, BAMLH0A0HYM2 with 52-week history for percentile ranks

### Added — Financial Stability tab (tab 10)

- Panel 1: NFCI vs ZQ EFFR path dual-axis chart with loose/tight shading
- Panel 2: Finance-neutral Taylor Rule (BIS/Borio), lfc slider, leaning premium badge
- Panel 3: Global dollar stress gauge (Fed swap lines + SOFR-EFFR basis)
- Panel 4: Doom loop scatter (NS term premium vs HY OAS), editable alert thresholds
- Panel 5: Financial cycle phase clock, 5-axis radar, composite stress index

### Academic references

- BIS Annual Report 2016 (Borio et al.) — finance-neutral r*, doom loop
- Mehrling (2016) "BIS looks through the financial cycle"
- Atlanta Fed MPT simplex regression methodology
- Wolfers & Zitzewitz (2004) — prediction markets as information aggregators

---

## [3.0] — March 2026

### Added — React migration & new analytical tabs

- **React architecture**: full rewrite from vanilla JS to React 18 (CDN, Babel JSX transform). `useReducer` for 17-action state machine, `useFinance` hook for derived computation (pure memoized pipeline). 9 tabs, all rendering from shared state — no globals.
- **Taylor Rule tab** (8th tab): five Taylor Rule variants with editable macro inputs and rule coefficients:
  - Taylor 1993 (α=1.5, β=0.5, GDP output gap)
  - Balanced approach / Yellen 2012 (α=1.0, β=1.0)
  - Yellen Alt1 (unemployment gap via Okun's Law, FOMC SEP r*)
  - Clarida-Gali-Gertler inertial (ρ=0.85 smoothing)
  - **Judd-Rudebusch ECM** (Bieri-Chincarini 2005 Eqs. 23–25): dynamic partial-adjustment specification with λ₁/λ₂/λ₃/γ parameters — designated **signal source** (highest R² for FFTR prediction, Exhibit 7)
  - SVG chart: prescribed rate paths (all 5 variants) vs CME market-implied path
  - Taylor−CME gap chart, meeting-by-meeting decomposition table, rule comparison matrix
  - GDP output gap / unemployment gap toggle with Okun's Law conversion
- **Yield Curve tab** (7th tab): Nelson-Siegel term structure modeling:
  - 7-tenor CMT input grid (3mo–30yr), short end synced from ZQ chain, long end editable or fetched from H.15
  - **Nelson-Siegel fitting**: Nelder-Mead optimizer over {β₀, β₁, β₂, τ} with 2000-iteration budget; RMSE badge in bp
  - NS factor cards: level, slope, curvature, decay, term premium
  - Curve chart (log-scale x-axis): raw CMT points, NS spot fit (amber), instantaneous forward (amber dashed), per-meeting scenario curves (amber dotted fan)
  - **Scenario shock propagation**: Bieri-Chincarini empirical factor loadings (2yr +0.50, 5yr +0.30, 10yr +0.10, 30yr +0.05 per 100bp FFTR change) applied cumulatively at each FOMC meeting
  - Shock impact table with maturity deltas, 10–2yr slope change, and riding signal per meeting
  - Fetch CMT rates button (Anthropic API + web search for Federal Reserve H.15)
- **Riding Calculator tab** (9th tab): Bieri-Chincarini (2005) riding-the-yield-curve implementation:
  - 7×5 HPR matrix: instruments (6mo–30yr) × horizons (1mo–18mo), each cell showing annualized XHPR (pp) and cushion (bp) with ride/marginal/hold coloring
  - **5 configurable filter rules**: positive slope, positive cushion, 75th percentile cushion, Taylor rule signal, CME expectations signal
  - Clickable cells with cushion detail panel: gauge bar with 75th percentile marker
  - Duration-neutral barbell panel: ω-weighted riding HPR vs overnight vs buy-and-hold decomposition (Eqs. 29–30)
  - Money-market HPR (Eq. 8) for m ≤ 1yr, zero-coupon bond HPR (Eq. 12) for m > 1yr
  - Break-even cushion: mmCushion (Eq. 15) and bondCushion (Eq. 16)
- **Dashboard tab** (6th tab): unified cross-market summary:
  - 9 stat cards: EFFR start, P(≥1 cut), expected terminal, expected cuts (CME·Kal·Poly), SOFR near-term/terminal, SOFR–EFFR basis, active scenario, **riding signal**
  - Unified rate path chart: CME EFFR chain + Kalshi implied + SR3 SOFR strip + Fed SEP + compare scenario on single canvas
  - Cross-market divergence table: all sources per meeting (CME rate, Kalshi rate, Kal−CME basis, P(cut) from all three, SOFR quarterly, basis bp)
  - Policy regime summary box with hike tail, emergency cut, Fed Chair regime-change risk
  - Scenario delta panel: compare vs active terminal rate and P(≥1 cut) spread
- **SOFR Strip tab** (5th tab): ported from v2 as React component (`SofrTab`) with editable SR3 prices dispatching through reducer
- **Cross-tab signal chain**: ZQ futures → EFFR chain → Taylor Rule (JR ECM signal) → Nelson-Siegel (scenario shocks) → Riding Calculator (filter cascade) → Dashboard (unified view). Changing any upstream input propagates through the entire chain via `useFinance` memoization.
- **Persistence**: `localStorage` saves lb, active scenario, and compare scenario across hard refreshes

### Added — Prediction markets (from earlier v3.0)

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
