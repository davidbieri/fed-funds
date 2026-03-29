# Claude Code Session Scripts
## MSE: A Multi-Market Framework for Federal Reserve Policy Analytics — Phases 1–4

> **Status: Phases 1–4 complete and deployed (v3.0).** This file is the implementation
> archive for the React migration (Phase 1), Taylor Rule tab (Phase 2), Yield Curve tab
> (Phase 3), and Riding Calculator tab (Phase 4). Do not modify — kept as historical record.
> Phase 5 (PCI + Financial Stability) and Phase 6 (Balance Sheet / QT + extended Financial
> Stability) scripts are maintained separately in the Claude project knowledge base.

---

## PHASE 1 — React Migration

### Context to paste at session start

```
I have a self-contained financial analytics tool at fed_funds_probability_tree.html (2,007 lines, 
single-file vanilla JS). I need it ported to React (CDN, no build step) as a new file 
fed_funds_v3.jsx, preserving every existing feature exactly.

Please start by reading the full file before writing any code.
```

### Step-by-step instructions

```
Read fed_funds_probability_tree.html in full. Then execute the following migration plan in order:

ARCHITECTURE:
- Single file: fed_funds_v3.jsx
- React 18 + ReactDOM via CDN (esm.sh or unpkg), Babel standalone for JSX transpilation
- No Tailwind — preserve the existing CSS variable design system verbatim as an inline <style> block
- State: one useReducer at App level, action types mirror every current mutation

STEP 1 — Extract and verify the pure math layer first.
Copy these functions verbatim into a // === MATH LAYER === section at the top of the script tag.
Do NOT modify their logic — just copy:
  fwChain(months)
  fwMeetings(months, chain)
  fwDist(meetings)
  getCompareSeries(lb)          [needs fwD, fwCompareScenario, SCENARIOS as arguments — refactor signature]
  cmeCutCountDist(final)        [needs fwD as argument — refactor]
  getMeetingComps(meetings)     [needs kalshiD, polyD as arguments — refactor]
  getCmeHikeProb(meetings)
  enumeratePaths(meetings, depth)
  zqQuarterlyAvg()              [needs fwD, srD as arguments — refactor]
  srInstFwd()                   [needs srD as argument — refactor]
  ycSyncShortEnd()              [needs fwD, ycD as arguments — refactor]
Run node --check on the extracted block before proceeding.

STEP 2 — Define the reducer.
Initial state shape:
{
  fwActive: 'inputs',
  fwLb: 3.50,
  fwActiveScenario: '',
  fwCompareScenario: '',
  fwD: [...],        // 12-row ZQ array (copy seed data verbatim)
  kalshiD: {...},    // copy seed data verbatim
  polyD: {...},      // copy seed data verbatim
  srD: [...],        // 7-row SR3 array
  ycD: [...],        // 7-row CMT array
}

Action types:
  SET_TAB, SET_LB, SET_SCENARIO, SET_COMPARE, DEACTIVATE_SCENARIO,
  UPDATE_ZQ_PRICE, UPDATE_ZQ_MTG, UPDATE_ZQ_N, UPDATE_ZQ_M,
  UPDATE_SR_PRICE, UPDATE_YC_RATE,
  APPLY_LIVE_ZQ, APPLY_LIVE_KALSHI, APPLY_LIVE_POLY, APPLY_LIVE_SOFR, APPLY_LIVE_YC,
  LOAD_SETTINGS

STEP 3 — useFinance(state) custom hook.
Wraps all derived computation in useMemo. Returns:
  { chain, meetings, states, final, compareSeries, zqAvgs, instFwd }
All math functions from Step 1 are called here with state slices as arguments.

STEP 4 — Port all async fetch functions.
  refreshData()    → useCallback, dispatches APPLY_LIVE_ZQ
  fetchKalshi()    → useCallback, dispatches APPLY_LIVE_KALSHI
  fetchPolymarket() → useCallback, dispatches APPLY_LIVE_POLY
  fetchSOFR()      → useCallback, dispatches APPLY_LIVE_SOFR
  fetchYieldCurve() → useCallback, dispatches APPLY_LIVE_YC
Each takes (state, dispatch, apiKey) as arguments. API key lives in local component state 
(not the reducer) since it is never persisted.

STEP 5 — Port all SVG helper functions as pure functions.
Each takes explicit arguments (no globals). Return SVG string (not JSX — keep as 
dangerouslySetInnerHTML for now, React SVG migration is Phase 2+).
  panel1Svg(comps, lb)
  panel2Svg(cme, kal, poly)
  panel4Svg(meetings, states, lb, compareSeries)   ← add compareSeries arg
  panel5Svg(cmeHike, kalHike, largeCut, polyHike)
  panel6Svg(meetings, states, lb, kalshiD, polyD)
  panel7Svg(final, lb, polyD)
  panel8aHtml(polyD)
  panel8bSvg(emergencyCut, polyHike)
  sofrMainChart(zqAvgs, instFwd, lb, srD, kalshiD)
  basisBarsSvg(zqAvgs, srD)
  dashUnifiedSvg(chain, meetings, states, lb, compareSeries, zqAvgs, kalshiD, srD)
  ycCurveChart(ycD)

STEP 6 — Port all tab renderer functions as React components.
Each receives props from the hook output + state slices. Use dangerouslySetInnerHTML 
for SVG output from Step 5.
  <FuturesInputs />     — scenario chips, compare picker, ZQ table, lb input
  <ChainTab />          — EFFR chain table + per-meeting breakdown
  <TreeTab />           — lattice + final distribution with compare overlay
  <KalshiTab />         — fetch bar + 9 panels
  <SofrTab />           — fetch bar + contract table + charts
  <DashboardTab />      — stat bar + unified chart + cross-market table + regime box
  <YieldCurveTab />     — fetch bar + CMT table + curve chart

STEP 7 — App shell.
  - Header (title, EFFR badge, target, data badge, API key input, Refresh ZQ button)
  - Tab bar (7 tabs, fwActive drives active class)
  - Tab panels (conditional render based on fwActive)
  - localStorage persistence: useEffect saves {lb, scenario, compare} on state change;
    LOAD_SETTINGS dispatched on mount

STEP 8 — Verification.
After writing the full file, run:
  node --check fed_funds_v3.jsx
If any syntax errors, fix before stopping. 
Then list all tab names and confirm each has a corresponding component rendered.

OUTPUT: A single file fed_funds_v3.jsx that can be opened directly in a browser 
(Babel transpiles inline). Also update index.html to point to fed_funds_v3.jsx.
```

### What to watch for / common failure modes

```
- The three chain solver passes (fwChain) use mutation — in React these must be 
  wrapped in useMemo so they don't run on every render. Flag any direct fwD mutation.
- cmeCutCountDist creates a secondary fwD filter internally — refactor to accept 
  fwD as an argument rather than closing over the global.
- The onchange handlers in the ZQ table currently dispatch fwRender() directly — 
  in React these become dispatch(UPDATE_ZQ_PRICE, {i, value}) actions.
- The SVG helpers use template literal strings — keep them as strings returned by 
  pure functions and injected via dangerouslySetInnerHTML. Do NOT attempt to convert 
  SVG strings to JSX in this phase — that's a separate pass.
- DO NOT add new features or change any financial logic. This phase is a pure port.
```

---

## PHASE 2 — Taylor Rule Tab

### Context to paste at session start

```
I have fed_funds_v3.jsx (the React port of my Fed Funds probability tree tool). 
I need to add a new Taylor Rule tab. Please read the file before writing any code.
The relevant background is:
- Atlanta Fed Taylor Rule Utility: https://www.atlantafed.org/research-and-data/data/taylor-rule
- Bieri-Chincarini (2005) "Riding the Yield Curve" for the Judd-Rudebusch ECM spec
  (Equations 22-25 in that paper define the dynamic Taylor rule we need)
```

### Step-by-step instructions

```
Read fed_funds_v3.jsx in full, then implement the Taylor Rule tab as follows:

STEP 1 — trD data object.
Add to initial reducer state:
{
  trD: {
    // Current macro inputs (user-editable, fetchable)
    inflationRate: 2.8,      // core PCE 4-quarter (%)
    inflationTarget: 2.0,    // π* (%)
    outputGap: -0.5,         // % of potential GDP (negative = slack)
    unemploymentRate: 4.1,   // current u-rate (%)
    unemploymentNatural: 4.1,// u* NAIRU from CBO (%)
    naturalRate: 0.5,        // r* real neutral rate from Laubach-Williams (%)
    // Rule parameters (user-adjustable via sliders)
    rhoSmoothing: 0.0,       // ρ inertia weight (0=no smoothing, 0.85=high)
    gapWeight: 0.5,          // β coefficient on resource gap
    gapMeasure: 'output',    // 'output' | 'unemployment'
    inflationMeasure: 'core_pce', // 'core_pce' | 'pce' | 'gdp_deflator'
    ruleVariant: 'taylor93', // 'taylor93' | 'yellen_alt1' | 'balanced' | 'judd_rudebusch'
    // Judd-Rudebusch ECM parameters (Equations 23-24)
    jrGamma: 0.85,           // speed of adjustment λ (partial adjustment weight)
    jrLambda1: 1.5,          // weight on inflation deviation
    jrLambda2: 0.5,          // weight on output gap
    jrLambda3: 0.3,          // weight on lagged output gap
    // Fetched state
    fetched: false, asOf: null, source: 'Seed data'
  }
}

STEP 2 — MATH: Taylor rule engine functions.
Add to the MATH LAYER section. These are pure functions, no DOM or state dependencies.

// taylorRate(trD, prevFFR) → prescribed rate (%)
// Implements Atlanta Fed generalized form:
//   FFR̂_t = ρ·FFR_{t-1} + (1-ρ)·[(r* + π*) + 1.5·(π - π*) + β·gap]
// gap = outputGap if gapMeasure='output', else 2×(u - u*) if 'unemployment'
function taylorRate(trD, prevFFR)

// taylorRateJR(trD, prevFFR, prevGap) → prescribed rate (%)  
// Judd-Rudebusch ECM (Equations 23-25 from Bieri-Chincarini 2005):
//   i*_t = r* + π* + λ1·(π_t - π*) + λ2·gap_t + λ3·gap_{t-1}
//   Δi_t = γ·(i*_t - i_{t-1}) + (1-γ)·Δi_{t-1}
function taylorRateJR(trD, prevFFR, prevGap)

// taylorSignal(prescribed, actual) → +1 | -1 | 0
// +1 = rule prescribes hike (riding less favorable)
// -1 = rule prescribes cut  (riding more favorable)
//  0 = within 12.5bp threshold (neutral)
function taylorSignal(prescribed, actual)

// taylorImpliedPath(trD, fwD, lb) → Array[12] of prescribed rates
// Projects the Taylor rule prescription over the 12-month ZQ horizon by 
// applying the rule at each FOMC meeting, using the CME-implied path as 
// the prevFFR input for the smoothing term.
function taylorImpliedPath(trD, fwD, lb)

STEP 3 — ADD to useFinance hook:
  taylorPrescribed: taylorRate(state.trD, state.fwLb)
  taylorSignalVal: taylorSignal(taylorPrescribed, currentEFFR)
  taylorImplied: taylorImpliedPath(state.trD, state.fwD, state.fwLb)
  taylorPolicyGap: (taylorPrescribed - currentEFFR) * 100   // in bp
These are derived state — include in useMemo dependencies on [trD, fwD, fwLb].

STEP 4 — fetchTaylorData() async function.
Prompt asks for: current core PCE (4-quarter), PCE, GDP deflator, unemployment rate, 
CBO NAIRU, Laubach-Williams r* (NY Fed), output gap (CBO). 
Returns JSON shape:
{
  "asOf": "YYYY-MM-DD", "source": "BEA/CBO/NY Fed",
  "inflationRate": 2.8, "inflationPCE": 3.1, "inflationGDP": 2.6,
  "unemploymentRate": 4.1, "unemploymentNatural": 4.1,
  "outputGap": -0.5, "naturalRate": 0.5
}
Dispatches APPLY_LIVE_TAYLOR action updating trD with fetched values.

STEP 5 — <TaylorRuleTab /> component. Three-panel layout:

Panel A (left, ~35% width) — Parameter controls:
  - Rule variant selector: four named chips (Taylor 1993 / Yellen Alt1 / Balanced / Judd-Rudebusch)
  - Sliders (range inputs with live numeric display):
    · Inflation target π*: 0–4%, step 0.25, default 2.0
    · Natural rate r*: 0–4%, step 0.25, default 0.5  
    · Gap weight β: 0–2.0, step 0.25, default 0.5
    · Smoothing ρ: 0–0.95, step 0.05, default 0.0
    · Gap measure toggle: GDP output gap / Unemployment gap (2×)
  - Fetch bar with fetchTaylorData button + current data source badge
  - 3×2 stat grid: prescribed rate, actual EFFR, policy gap (bp), Taylor signal, 
    inflation input, gap input

Panel B (center, ~40% width) — Multi-line chart:
  SVG with 4 series over the 12-month ZQ horizon:
  · CME implied path (blue, solid) — from useFinance chain
  · Taylor rule prescribed path (charcoal, solid) — from taylorImplied  
  · Compare scenario path (amber, dashed) — from compareSeries if active
  · Fed SEP dots (purple circles) — from kalshiD.fedDots
  Vertical "now" line at month 0. Y-axis: rate in %.
  Below chart: a 2-row basis table — for each FOMC meeting:
    | Meeting | CME implied | Taylor prescribed | Taylor−CME basis (bp) | Signal |

Panel C (right, ~25% width) — Rule comparison matrix:
  Table showing all 4 rule variants side by side at current inputs:
  | Rule | Prescribed | Gap (bp) | Signal |
  | Taylor 1993 | X.XX% | ±XXbp | ↓/↑/= |
  | Yellen Alt1 | X.XX% | ±XXbp | ↓/↑/= |
  | Balanced | X.XX% | ±XXbp | ↓/↑/= |
  | Judd-Rudebusch | X.XX% | ±XXbp | ↓/↑/= |
  Below: TaylorSignal indicator card — large colored badge (green=cut, red=hike, gray=neutral)
  Note: "Signal feeds riding calculator filter in Phase 4"

STEP 6 — Wire taylorImplied into Panel 4 (Kalshi dot plot).
In panel4Svg(), add the taylorImplied series as a fifth line (charcoal, short-dash).
Update the panel note to mention Taylor rule series.

STEP 7 — Wire taylorSignalVal into Dashboard regime box.
Add "Taylor rule signal" row to the regime summary panel.

STEP 8 — Add 'taylor' to tab routing (SET_TAB reducer, fwSwitch equivalent).
Tab label: "📏 Taylor Rule"

STEP 9 — Syntax check, then confirm:
  - taylorRate() returns a number between 0 and 15 for reasonable inputs
  - taylorSignal() returns exactly +1, -1, or 0
  - taylorImpliedPath() returns an array of length 12
  - The tab renders without errors when trD is at seed values
```

---

## PHASE 3 — Yield Curve / Term Structure Tab

### Context to paste at session start

```
I have fed_funds_v3.jsx with the Taylor Rule tab completed. I need to add a full 
yield curve / term structure tab implementing Nelson-Siegel-Svensson fitting and 
a scenario impact analyzer. The ycD data object (7 maturity points, 3mo–30yr) is 
already in state from the migration. 

Key references:
- Nelson-Siegel (1987) and Svensson (1994) parametric curve fitting
- Bieri-Chincarini (2005) Exhibit 5: Δslope_{10-2} ≈ -0.25 × ΔFFTR (per 100bp)
- Wu (2001/2003): slope factor explains 80-90% of yield curve variation from 
  monetary policy shocks
Please read the file before writing any code.
```

### Step-by-step instructions

```
Read fed_funds_v3.jsx in full, then implement the yield curve tab as follows:

STEP 1 — MATH: Nelson-Siegel fitting engine.
Add these pure functions to the MATH LAYER section:

// nsYield(mat, params) → rate (%)
// Nelson-Siegel spot rate for maturity `mat` (in years):
//   y(m) = β0 + β1·[(1-exp(-m/τ))/(m/τ)] + β2·[(1-exp(-m/τ))/(m/τ) - exp(-m/τ)]
// params = {beta0, beta1, beta2, tau}
function nsYield(mat, params)

// nsForward(mat, params) → rate (%)
// Instantaneous forward rate:
//   f(m) = β0 + β1·exp(-m/τ) + β2·(m/τ)·exp(-m/τ)
function nsForward(mat, params)

// nsFit(points) → {beta0, beta1, beta2, tau, rmse}
// Nelder-Mead minimization of sum-squared residuals over 4 parameters.
// points = [{mat, rate}] — the 7 ycD points
// Constraints: beta0 > 0, beta0+beta1 > 0 (positive short rate), tau > 0.1
// Initial simplex: [[4,−1,0.5,2], [5,−2,1,1.5], [3,−0.5,1,3], [4,−1.5,0.5,1], [4,−1,2,2]]
// Termination: 2000 iterations max or SSR < 1e-8
// Returns best params + RMSE in basis points
function nsFit(points)

// nsSvensson(mat, params) → rate (%)
// Extended Svensson form (optional toggle) — adds second hump term:
//   y(m) = β0 + β1·f1 + β2·f2 + β3·f3  where f3 = [(1-exp(-m/τ2))/(m/τ2) - exp(-m/τ2)]
// params = {beta0, beta1, beta2, beta3, tau1, tau2}
function nsSvensson(mat, params)

// nsShockCurve(ycD, deltaFFR, meetingIndex) → ycD-shaped array with shocked rates
// Applies the Bieri-Chincarini empirical factor model to project yield curve 
// impact of a funds rate change at a given meeting:
//   Δy_2yr  ≈ +0.50 × ΔFFR   (short end moves with funds rate)
//   Δy_5yr  ≈ +0.30 × ΔFFR   (intermediate partial pass-through)  
//   Δy_10yr ≈ +0.10 × ΔFFR   (long end less responsive)
//   Δy_30yr ≈ +0.05 × ΔFFR   (very long end nearly anchored)
// Coefficients from Exhibit 5 regression + standard literature priors.
function nsShockCurve(ycD, deltaFFR)

STEP 2 — ADD to useFinance hook:
  nsFitParams: useMemo(() => nsFit(state.ycD), [state.ycD])
  nsCurve: array of 200 points from 0.08yr to 30yr using nsYield(m, nsFitParams)
  nsForwardCurve: same 200 points using nsForward(m, nsFitParams)
  // Scenario-shocked curves — one per FOMC meeting
  scenarioCurves: meetings.map((m, i) => {
    const deltaFFR = (chain[i].end - chain[i].start) * 100  // cumulative bp change
    return { meeting: m.lb, curve: nsShockCurve(state.ycD, cumulativeDelta), params: nsFit(shocked) }
  })
  // NS factor interpretation
  nsFactors: {
    level: nsFitParams.beta0,          // long-run rate
    slope: nsFitParams.beta1,          // short-minus-long spread
    curvature: nsFitParams.beta2,      // hump magnitude
    decayParam: nsFitParams.tau
  }

STEP 3 — <YieldCurveTab /> component. Two-panel layout:

Panel A (full width) — Current curve + NS fit + scenario animator:

  Sub-panel A1: Curve chart (SVG, 720×180)
  Five overlaid series:
    · Raw data points: blue circles (ZQ-derived 3mo–1yr), green circles (CMT 2yr–30yr)
    · Linear interpolation between raw points: thin gray line
    · NS fitted spot curve: amber solid line (200 evaluation points, 0.08–30yr)
    · NS instantaneous forward curve: amber dashed line
    · Scenario-shocked curve: one colored line per active scenario meeting
      (use fwD meeting colors: step through blue→red spectrum)
  
  Sub-panel A2: NS factor cards (4 stat cards inline below chart):
    | Level (β₀) | Slope (β₁) | Curvature (β₂) | Decay τ |
    | X.XX% | −X.XX% | +X.XX% | X.XX yr |
    RMSE badge: "Fit RMSE: X.X bp"
  
  Sub-panel A3: Scenario animator controls:
    · Meeting selector: radio or segmented control over the 8 FOMC meetings
    · When a meeting is selected: show the shocked curve for that meeting overlaid
      in amber, alongside a stat showing the cumulative Δ from current
    · "Show all meetings" toggle: overlays all 8 shocked curves as a gradient 
      from light to dark (opacity 0.15 per line, darkening toward final meeting)

Panel B (full width, below A) — Shock impact table:

  Title: "Yield curve impact by FOMC meeting — Bieri-Chincarini empirical coefficients"
  Table: one row per FOMC meeting, columns:
    | Meeting | EFFR Δ (bp) | 2yr Δ (bp) | 5yr Δ (bp) | 10yr Δ (bp) | 30yr Δ (bp) | 
    | 10-2yr slope Δ (bp) | Riding signal |
  
  Color coding: green = curve steepening (favorable for riding), 
                red = curve flattening/inverting (unfavorable)
  
  Final row: "Cumulative (all meetings)" — sum of all scenario deltas.
  
  Note below table: "Slope Δ per 100bp hike ≈ −25bp (Bieri-Chincarini Exhibit 5, 
  1982–2003 U.S. Treasuries). Empirical coefficients by maturity derived from 
  Wu (2003) VAR: slope factor explains 80–90% of policy shock transmission."

STEP 4 — Wire nsYield into riding calculator prep.
Export nsYield and nsFitParams from useFinance so the riding tab (Phase 4) can call:
  y_mh = nsYield(m - h, nsFitParams)  for bond-market rides
This is the key cross-tab data dependency. Just expose it in the hook return — 
no UI change needed.

STEP 5 — Add Svensson toggle.
In Panel A controls, add a small "Model: NS / NSS" toggle. When NSS is selected:
- Show 6-parameter form (β₀–β₃, τ₁, τ₂)
- Refit using nsSvensson
- Display additional factor card for β₃
Default to NS (4-parameter) — NSS only adds value for humped curves.

STEP 6 — Update Dashboard unified chart.
Wire the NS fitted curve endpoint (30yr rate) into the Dashboard as an additional 
stat card: "Term premium proxy" = β₀ − current EFFR (the spread between the 
long-run level factor and the short rate).

STEP 7 — Syntax check and verify:
  - nsFit() converges for ycD seed data (rates roughly 3.6–4.6%)
  - nsYield(0.25, params) ≈ ycD[0].rate within 10bp
  - nsYield(10, params) ≈ ycD[5].rate within 20bp
  - scenarioCurves has length equal to number of active FOMC meetings
  - The tab renders without errors at seed ycD values
```

---

## PHASE 4 — Riding the Yield Curve Calculator Tab

### Context to paste at session start

```
I have fed_funds_v3.jsx with the Taylor Rule and Yield Curve tabs completed.
I need to add the riding the yield curve calculator based on Bieri and Chincarini 
(2005) "Riding the Yield Curve: A Variety of Strategies," Journal of Fixed Income.

The paper defines the exact formulas I need. Key equations:
  Eq. 8/9:   Money-market HPR and excess HPR
  Eq. 12/13: Bond-market HPR and excess HPR  
  Eq. 14:    Duration approximation for bond HPR
  Eq. 15/16: Break-even rate / cushion
  Eq. 29/30: Duration-neutral barbell weight and excess return

The tool already has:
  - ycD: 7-point yield curve data
  - nsFitParams + nsYield(mat, params): NS curve for any maturity
  - taylorSignalVal: +1/-1/0 riding signal from Taylor rule tab
  - fwD chain: CME futures signal (implied vs actual EFFR)
  - fwLb: current EFFR lower bound

Please read the full file before writing any code.
```

### Step-by-step instructions

```
Read fed_funds_v3.jsx in full, then implement the riding calculator tab:

STEP 1 — MATH: Riding calculator engine.
Add these pure functions to the MATH LAYER section.
All rates are in decimal (4.31% = 0.0431). Day-count z = 360 for money market, 
365 for bonds. Maturity m and holding horizon h in years.

// mmHPR(ym, ymh, m, h) → holding period return (decimal)
// Eq. 8: Money-market ride HPR
//   H = [1 + y_{m-h}·(m-h)·z] / [1 + y_m·m·z] - 1
// ym   = current rate for m-maturity instrument
// ymh  = projected rate for (m-h)-maturity instrument at end of holding period
// z    = 360 (money-market day count in years, so m·z gives days/360 fraction)
function mmHPR(ym, ymh, m, h, z=360)

// bondHPR(ym, ymh, m, h) → holding period return (decimal)
// Eq. 12: Bond (zero-coupon) ride HPR
//   H = [(1 + y_m)^m / (1 + y_{m-h})^{m-h}] - 1
// For coupon-paying bonds use duration approximation (Eq. 14):
//   H ≈ y_m · h - Δy · D_{m-h}   where Δy = ymh - ym, D = modified duration
function bondHPR(ym, ymh, m, h)

// mmCushion(ym, yh, m, h) → break-even yield change (%, positive = room to rise)
// Eq. 15: break-even rate for money-market ride
//   y* = z·[(1 + ym·m/z) / (1 + yh·h/z) - 1] / (m-h)
//   cushion = y* - current y_{m-h}
// yh = current rate for h-maturity (the benchmark buy-and-hold rate)
function mmCushion(ym, yh, ymh, m, h, z=360)

// bondCushion(ym, yh, ymh, m, h) → break-even yield change (%)
// Eq. 16: break-even rate for bond ride
//   y* such that XH = 0; solved from: (1+ym)^m / (1+y*)^{m-h} = (1+yh)^h
//   cushion = y* - ymh
function bondCushion(ym, yh, ymh, m, h)

// durationNeutralWeight(dH, dR, dO=0) → ω (decimal weight on riding instrument)
// Eq. 29: ω = (D_H - D_O) / (D_R - D_O)
// dH = duration of holding horizon (h in years ≈ h for zero-coupon)
// dR = duration of riding instrument (m in years ≈ m for zero-coupon)
// dO = duration of overnight deposit (0)
function durationNeutralWeight(dH, dR, dO=0)

// durationNeutralXR(hRide, hOvernight, hBuyHold, omega) → excess return (decimal)
// Eq. 30: XR = ω·H_ride + (1-ω)·H_overnight - H_buyhold
function durationNeutralXR(hRide, hOvernight, hBuyHold, omega)

// rideMatrix(ycD, nsFitParams, fwD, fwLb, horizon_h, filters) → 7×5 matrix
// Computes HPR, XHPR, cushion, duration-neutral XR for all instrument×horizon combos.
// Instrument maturities (rows): 0.5, 1, 2, 5, 7, 10, 30 yr
// Holding horizons (cols): 1mo, 3mo, 6mo, 12mo, 18mo (in years: 1/12, 3/12, 6/12, 1, 1.5)
// 
// For each (m, h) cell:
//   ym    = nsYield(m, nsFitParams) — current rate from fitted curve
//   yh    = nsYield(h, nsFitParams) — benchmark (buy-and-hold) rate  
//   ymh   = scenarioYield(m-h, ...)  — projected rate at horizon (from active scenario)
//           For scenario projection: use nsYield on shocked curve at meeting closest to h
//           If no scenario: use forward rate implied by NS curve (nsForward(m-h, params))
//   HPR   = bondHPR(ym, ymh, m, h)  [use mmHPR if m <= 1yr]
//   XHPR  = HPR - bondHPR(yh, yh, h, h)  [vs buy-and-hold at h]
//   cushion = bondCushion(ym, yh, ymh, m, h)
//   ω     = durationNeutralWeight(h, m)
//   dnXR  = durationNeutralXR(HPR, overnight_rate/12*h, yh*h, ω)
//   signal = rideSignal(cushion, filters, taylorSignal, cmeFuturesSignal)
function rideMatrix(ycD, nsFitParams, scenarioCurves, fwD, fwLb, taylorSignalVal, filters)

// rideSignal(cushion, cushionHistory, filters, taylorSignal, cmeFuturesSignal) → 
//   'ride' | 'hold' | 'marginal'
// Applies the four Bieri-Chincarini filter rules (AND logic when multiple enabled):
//   1. positiveSlope:    yield curve slope (10yr-2yr) > 0
//   2. positiveCushion:  cushion > 0
//   3. cushion75:        cushion > 75th percentile of trailing 24-month distribution
//                        (approximated as cushion > 1.5× median when no history)
//   4. taylorRule:       taylorSignal !== +1 (rule not prescribing hike)
//   5. cmeExpectations:  CME-implied rate < current EFFR (market pricing a cut)
// Returns 'ride' if all enabled filters pass, 'hold' if any fail, 'marginal' if borderline
function rideSignal(cushion, slope, filters, taylorSignal, cmeFuturesSignal)

STEP 2 — ADD to useFinance hook:
  rideData: useMemo(() => rideMatrix(
    state.ycD, nsFitParams, scenarioCurves, state.fwD, state.fwLb, 
    taylorSignalVal, state.rideFilters
  ), [state.ycD, nsFitParams, scenarioCurves, state.fwD, state.fwLb, taylorSignalVal, state.rideFilters])
  
Add to initial reducer state:
  rideFilters: {
    positiveSlope: true,
    positiveCushion: true,
    cushion75: false,
    taylorRule: false,    // enabled automatically when Taylor tab is populated
    cmeExpectations: true
  }
  rideSelected: { matIdx: 3, hIdx: 1 }  // default: 2yr instrument, 3-month horizon
  
Add reducer actions: TOGGLE_RIDE_FILTER, SET_RIDE_SELECTED

STEP 3 — <RidingTab /> component. Three-panel layout:

Panel A (full width) — Instrument × horizon matrix:

  Header row: filter toggle chips (5 chips, one per filter, active = filled)
  "Active filters: [Positive slope ✓] [Positive cushion ✓] [75th pctile] [Taylor rule] [CME signal ✓]"
  Each chip: click to toggle the filter, updates rideFilters in reducer.

  Matrix table (7 rows × 5 cols + headers):
  Row headers: 6mo | 1yr | 2yr | 5yr | 7yr | 10yr | 30yr
  Col headers: 1mo | 3mo | 6mo | 12mo | 18mo
  
  Each cell: 
    · Background color: green (#dcfce7) = ride, red (#fee2e2) = hold, 
      yellow (#fef9c3) = marginal — based on rideSignal output
    · Line 1: XHPR formatted as "+X.XXpp" in bold (annualized, in percentage points)
    · Line 2: "cushion: XXbp" in small gray text
    · Clicking a cell: dispatches SET_RIDE_SELECTED, updates Panels B and C
    · Active cell: blue border highlight
  
  Below matrix: legend and note:
    "XHPR = annualized excess HPR vs buy-and-hold at holding horizon. 
    Cushion = break-even yield change (bp) — how much rates must rise to eliminate 
    excess return. Color = all enabled filter rules. Source: Bieri-Chincarini (2005) 
    equations 8-16."

Panel B (left half) — Cushion detail for selected ride:

  Title: "Cushion analysis — [2yr instrument, 3-month horizon]"  (updates with selection)
  
  Sub-panel B1: Cushion gauge bar
    · Horizontal bar: current cushion in bp vs a scale of −100bp to +200bp
    · 75th-percentile marker line
    · Color: green if above 75th pctile, amber if positive, red if negative
    · Large number: "Cushion: +XXbp"
    · Sub-text: "Rates must rise XX bp to eliminate excess return"
  
  Sub-panel B2: Four-way comparison table for selected (m, h):
    | Metric | Raw ride | Duration-neutral |
    | HPR    | X.XX%    | X.XX%           |
    | XHPR   | +X.XXpp  | +X.XXpp         |
    | Sharpe | X.XX     | X.XX            |
    | Signal | RIDE / HOLD | RIDE / HOLD  |
  
  Sub-panel B3: Taylor-conditioned note
    If taylorSignalVal = -1: "Taylor rule prescribes cut → riding favorable ✓"
    If taylorSignalVal = +1: "Taylor rule prescribes hike → riding less favorable ✗"
    If taylorSignalVal = 0:  "Taylor rule neutral → no additional signal"
    If Taylor tab not populated: "Enable Taylor Rule tab for conditioning signal"

Panel C (right half) — Duration-neutral riding view:

  Title: "Duration-neutral barbell — selected ride"
  
  Sub-panel C1: Barbell weight display
    Visual: two bars side by side
    · Left bar:  "ω = XX% in [2yr instrument]"  (riding weight)
    · Right bar: "1-ω = XX% in overnight"        (cash weight)
    Formula display: "ω = (D_H − D_O) / (D_R − D_O) = (Xmo − 0) / (Xyr − 0)"
  
  Sub-panel C2: Return breakdown
    Three-row table:
    | Component | Return |
    | Riding instrument (ω×HPR) | +X.XXpp |
    | Overnight deposit ((1-ω)×r_o) | +X.XXpp |
    | Buy-and-hold (h-maturity) | −X.XXpp |
    | Duration-neutral XR | +X.XXpp |
    
    Duration target: "Targeting duration of X-month buy-and-hold = X.XX years"
  
  Sub-panel C3: Government vs credit comparison (qualitative)
    Static reference card (seed data, no live fetch needed):
    "Credit premium for USD swaps vs Treasuries: ~15–30bp (TED spread analog).
    At current SOFR–EFFR basis of +Xbp, LIBOR/swap ride adds approximately
    X–Xpp annualized excess return vs Treasury ride (Bieri-Chincarini Eq. 20–21)."
    This is a qualitative annotation — full credit ride calculation is Phase 5 scope.

STEP 4 — Wire riding signal into Dashboard.
Add "Riding signal" stat card to the Dashboard summary bar:
  · Value: "RIDE" (green) / "HOLD" (red) / "—" (gray)
  · Sub-text: "2yr / 3mo · X filters active"
  · Based on rideData[3][1].signal (the 2yr × 3mo default cell)

STEP 5 — Wire riding signal into Futures Inputs tab.
Below the scenario chips, add a single-line riding summary:
  "Current riding signal (2yr/3mo): [RIDE ✓] · Cushion: +XXbp · Taylor: ↓"
This updates reactively when scenario changes because rideData is in useMemo.

STEP 6 — Syntax check and verify:
  - rideMatrix returns a 7×5 nested array, no nulls for standard curve shapes
  - mmHPR(0.0355, 0.032, 0.5, 0.25) ≈ +0.5% (roughly — sanity check only)
  - bondHPR(0.043, 0.038, 10, 0.25) > bondHPR(0.043, 0.040, 10, 0.25) 
    (lower end rate → higher riding return ✓)
  - durationNeutralWeight(0.25, 2) = 0.125 (25% of 2yr duration)
  - All filter toggles update matrix colors without page errors
  - SET_RIDE_SELECTED updates Panels B and C correctly

ADDITIONAL NOTES FOR ALL PHASES:

Performance:
  - nsFit() (Nelder-Mead, 2000 iterations) runs in <15ms for 7 points.
    Wrap in useMemo with ycD as dependency — only reruns when rates change.
  - rideMatrix (7×5 = 35 cells × a few function calls each) is trivially fast.
  - No web workers needed at this scale.

GitHub deployment:
  After each phase, update CHANGELOG.md with the new tab name and key formulas 
  implemented. The file can be pushed to github.com/davidbieri/fed-funds as a 
  drop-in replacement for the existing HTML file (index.html redirect unchanged).

Regression prevention:
  After each phase, manually verify these things still work:
  1. ZQ price edit in Futures Inputs updates the probability tree
  2. Scenario chip apply/deactivate works
  3. Compare overlay shows amber on Panel 4 and final distribution
  4. Dashboard cross-market table populates
  5. SOFR basis bars show positive values
  If any of these break, fix before adding new features.
```
