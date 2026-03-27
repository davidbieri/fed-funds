# MSE 

**A multi-market framework for Federal Reserve policy analytics.**

[![Version](https://img.shields.io/badge/version-3.0-blue)](https://github.com/davidbieri/fed-funds/blob/main/CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub Pages](https://img.shields.io/badge/deployed-GitHub%20Pages-brightgreen)](https://davidbieri.github.io/fed-funds/)

**Live tool:** [davidbieri.github.io/fed-funds](https://davidbieri.github.io/fed-funds/)  
**Virginia Tech · School of Public and International Affairs**

> *MSE — after Marriner S. Eccles (1890–1977), principal economic philosopher of the New Deal, architect of the modern Federal Reserve.*

> For educational and research purposes only. Not financial advice.

---

## Overview

MSE implements the CME FedWatch methodology from first principles — deriving per-meeting cut/hold/hike probabilities from 30-day Fed Funds (ZQ) futures prices — and extends it with four layers of analytical context: prediction market comparison (Kalshi and Polymarket), SOFR strip basis analysis, Nelson-Siegel term structure modeling, Taylor Rule prescriptive analysis, and a complete Bieri-Chincarini (2005) riding-the-yield-curve calculator.

All nine tabs share a single state machine. Changing any upstream input — a ZQ price, a macro variable, a scenario preset — propagates instantly through the entire `useFinance` memoization pipeline to every downstream tab.

No build step. No npm. Open directly in a browser or deploy as a static file.

---

## Nine tabs

| # | Tab | Description |
|---|-----|-------------|
| 01 | **Futures Inputs** | Editable ZQ price table (Apr 2026–Mar 2027), FOMC meeting flags, N/M day splits. Source of truth for all tabs. |
| 02 | **EFFR Chain** | EFFR anchor propagation: non-meeting months fix EFFR(End) = EFFR(Avg) = EFFR(Start+1); meeting months solved via the day-weighted FedWatch formula. |
| 03 | **Probability Tree** | SVG unconditional probability lattice across all FOMC meetings. Node opacity ∝ probability. Summary stats: P(≥1/2/3 cuts), expected cuts, implied rate, final distribution. |
| 04 | **Prediction Markets** | Nine-panel comparison: CME vs Kalshi vs Polymarket. Meeting divergence, cut count distributions, combo paths, dot plots, hike risk, first-cut timing, terminal rates, Fed Chair regime risk, emergency cut gauge. |
| 05 | **SOFR Strip** | 7 quarterly SR3 contracts (SR3M6–SR3Z7, Jun 2026–Dec 2027). Three-curve chart: SOFR strip, ZQ quarterly EFFR avg, instantaneous 3M forward. Per-quarter SOFR–EFFR basis in bp. |
| 06 | **Dashboard** | Unified cross-market view: 9 stat cards, combined rate path chart (CME + Kalshi + SOFR + SEP), cross-market divergence table, policy regime summary, riding signal. |
| 07 | **Yield Curve** | Nelson-Siegel fitting over 7 Treasury CMT maturities (3mo–30yr) via 2,000-iteration Nelder-Mead. Log-scale spot and instantaneous forward curves. Scenario-shocked paths per FOMC meeting using Bieri-Chincarini empirical factor loadings. |
| 08 | **Taylor Rule** | Five rule variants: Taylor 1993, Balanced/Yellen 2012, Yellen Alt1 (Okun's Law), Clarida-Gali-Gertler inertial (ρ=0.85), Judd-Rudebusch ECM (Bieri-Chincarini 2005, Eqs. 23–25). Prescribed vs CME path chart, Taylor−CME gap, per-meeting rule matrix. |
| 09 | **Riding** | Bieri-Chincarini (2005) riding-the-yield-curve calculator: 7×5 HPR matrix (6mo–30yr instruments × 1mo–18mo horizons), excess HPR and break-even cushion in bp, five-filter cascade, duration-neutral barbell decomposition. |

---

## Methodology

### CME FedWatch (tabs 01–03)

The tool replicates the official CME FedWatch methodology from first principles.

**Non-meeting months (anchor rule):**

$$\text{EFFR}(\text{End})_{T-1} = \text{EFFR}(\text{Avg})_{T} = \text{EFFR}(\text{Start})_{T+1}$$

**Meeting months:**

$$\text{EFFR}(\text{Start}) = \frac{\text{EFFR}(\text{Avg}) - \dfrac{M}{N+M} \cdot \text{EFFR}(\text{End})}{\dfrac{N}{N+M}}$$

where *N* = pre-meeting days, *M* = post-meeting days in the contract month.

The expected rate change EFFR(End) − EFFR(Start) is divided by 25bp to yield a fractional move count. The characteristic integer gives the base move; the mantissa gives P(additional 25bp move).

**Unconditional lattice:**

$$P(\text{node at meeting } M, \text{ rate } R) = \sum_{R'} P(\text{prior node at } R') \times P(R' \to R \mid M)$$

### Nelson-Siegel term structure (tab 07)

$$y(\tau) = \beta_0 + \beta_1 \frac{1 - e^{-\tau/\lambda}}{\tau/\lambda} + \beta_2 \left(\frac{1 - e^{-\tau/\lambda}}{\tau/\lambda} - e^{-\tau/\lambda}\right)$$

Parameters {β₀, β₁, β₂, λ} are estimated via Nelder-Mead minimization of RMSE across the 7 CMT input tenors (2,000-iteration budget). Instantaneous forward rate derived analytically from the NS spot curve. Scenario shocks applied using Bieri-Chincarini (2005) empirical factor loadings: 2yr +0.50, 5yr +0.30, 10yr +0.10, 30yr +0.05 per 100bp FFTR shift.

### Riding the yield curve (tab 09)

Zero-coupon excess holding-period return (Bieri-Chincarini 2005, Eq. 12):

$$\text{XHPR}(m, h) = \text{HPR}(m, h) - \text{mmHPR}(h)$$

Break-even cushion (Eqs. 15–16):

$$\text{Cushion}(m,h) = \frac{y(m) - y(h)}{D(m,h)} \times 100 \text{ bp}$$

Duration-neutral barbell weight ω (Eq. 29) and riding HPR decomposition (Eq. 30) are computed in the barbell panel. The aggregate riding signal feeds the Dashboard stat card.

---

## Data sources

| Source | Data | Refresh |
|--------|------|---------|
| **CME** | ZQ 30-day Fed Funds futures (Apr 2026–Mar 2027) | Refresh ZQ button |
| **CME** | SR3 3-month SOFR futures (SR3M6–SR3Z7) | Fetch SR3 button |
| **Kalshi** | Per-meeting decision markets, cut count, hike risk, jumbo cut, implied rates | Fetch Kalshi button |
| **Polymarket** | Per-meeting probs, cut count (13 outcomes), terminal rate, Fed Chair nomination ($230M vol), emergency cut, first-cut timing | Fetch Polymarket button |
| **Federal Reserve** | EFFR (NY Fed daily), SOFR fixing, Dec 2025 SEP dot plot, H.15 CMT rates (7 tenors) | H.15 fetch via Yield Curve tab |

Live refresh uses the Anthropic API with web search (`claude-sonnet-4-6`). Inside claude.ai the API key is injected automatically. For standalone use, paste an `sk-ant-...` key into the field in the top-right corner of the tool.

---

## Prediction market panels (tab 04)

Nine panels comparing CME-derived probabilities against Kalshi and Polymarket:

1. **Meeting divergence** — 3 bars per meeting (CME / Kalshi / Polymarket) with Kal−CME and Poly−Kal basis pills
2. **Cut count distribution** — P(0–5+ cuts in 2026) across all three sources (Polymarket has 13 discrete strikes)
3. **Combo path decomposer** — Top 8 hold/cut path sequences vs Kalshi kxfedcombo contracts
4. **Market-implied dot plot** — CME chain + Kalshi implied rates + Fed Dec 2025 SEP on one chart
5. **Hike risk monitor** — Semicircular gauge with three concentric arcs (CME / Kalshi / Polymarket)
6. **First-cut timing** — CME step curve + Kalshi year-end circles + Polymarket per-meeting diamonds
7. **Terminal rate distribution** — CME final distribution vs Polymarket end-2026 market
8. **Fed Chair regime risk** — Polymarket $230M nominee market with hawkishness tags
9. **Emergency cut gauge** — P(unscheduled FOMC cut before end-2026), Polymarket-unique

The key analytical signal is the *basis* between markets — Kal−CME captures the regulated vs futures spread; Poly−Kal captures the information gap between institutional/domestic (Kalshi) and global/retail/crypto-native (Polymarket) sentiment.

---

## Scenario presets

Nine named ZQ price configurations activate with a single click and restore on second click:

| Scenario | Tag | Description |
|----------|-----|-------------|
| Warsh pivot | hawkish | Hawkish new chair; easing priced out ~50bp across strip |
| Emergency cut | tail risk | Unscheduled cut before May; front-end repriced sharply |
| Stagflation hold | hawkish | Oil shock + sticky inflation; no cuts through 2026 |
| June liftoff | base | First cut Jun 17–18; ~2 cuts in 2026 |
| Delayed easing | base | First cut Sep 16–17; ~1 cut in 2026 |
| Soft landing | base | 3 cuts evenly spaced through H2 2026 |
| Recession fast cuts | tail risk | 4+ cuts; aggressive easing from September |
| No cuts 2026 | hawkish | Fed on hold through entire 2027 horizon |
| Three cuts | base | Standard consensus path; ~75bp cumulative easing |

Activating a scenario saves your current manual prices. Clicking the active scenario again restores them exactly.

---

## Architecture

```
fed_funds_probability_tree.html   ← React 18 (CDN) application, ~3,500 lines
index.html                        ← Landing page / documentation wrapper
README.md
CHANGELOG.md
CLAUDE.md                         ← Claude Code session instructions
```

**React 18** via CDN with Babel JSX transform — no build step, no bundler.

**State**: `useReducer` with a 17-action state machine. All derived values computed in a `useFinance` hook (pure memoized pipeline). Nine tabs render from shared state — no globals.

**Signal chain**: ZQ prices → EFFR chain → Probability tree → Prediction markets overlay → SOFR strip → Taylor Rule (JR ECM signal) → Yield Curve (NS + scenario shocks) → Riding (HPR + filter cascade) → Dashboard (unified view).

**Persistence**: `localStorage` saves ZQ prices, active scenario, and compare scenario across hard refreshes.

---

## File structure

```
davidbieri.github.io/fed-funds/
├── index.html                    ← Landing page (this file launches tool in iframe modal)
└── fed_funds_probability_tree.html  ← The tool itself
```

---

## Usage

No build step, no dependencies. Open `fed_funds_probability_tree.html` directly in any modern browser.

```bash
git clone https://github.com/davidbieri/fed-funds.git
cd fed-funds
open fed_funds_probability_tree.html   # macOS
# or: xdg-open fed_funds_probability_tree.html  (Linux)
# or: start fed_funds_probability_tree.html      (Windows)
```

All values in the **Futures Inputs** tab are editable at any time — override any live-fetched price manually, adjust N/M day splits, or toggle which months contain FOMC meetings. Scenario presets can be activated with a single click and restored just as easily.

---

## GitHub Pages

The tool is live at **[davidbieri.github.io/fed-funds](https://davidbieri.github.io/fed-funds/)** — no login, no server, opens directly in any browser.

To enable Pages on a fork: **Settings → Pages → Source: Deploy from branch → Branch: main / (root) → Save**.

---

## Limitations

- Probabilities assume rate changes are uniformly sized in **25bp increments** and that EFFR reacts proportionally within the contract month.
- The EFFR is bounded below at 0% by convention (not binding in the current rate environment).
- Kalshi and Polymarket data fetched via web search may reflect slightly stale settlement prices depending on market hours.
- Nelson-Siegel fitting is unconstrained — in extreme curve regimes the optimizer may find a local minimum; the RMSE badge in the Yield Curve tab signals fit quality.
- Taylor Rule prescriptions are sensitive to the assumed r* and output/unemployment gap inputs; treat prescribed paths as directional signals, not point forecasts.
- This tool is for **research and educational purposes only** and does not constitute investment advice.

---

## Background

Developed as part of a broader research program connecting financial market microstructure, monetary policy, and geoeconomic analysis at Virginia Tech's School of Public and International Affairs. The CME FedWatch methodology was reverse-engineered from the [official CME documentation](https://www.cmegroup.com/articles/2023/understanding-the-cme-group-fedwatch-tool-methodology.html); the Kalshi and Polymarket comparison layers, the Nelson-Siegel term structure module, the Taylor Rule suite, and the riding-the-yield-curve calculator are original extensions grounded in the academic literature cited below.

---

## Citation

**APA**
> Bieri, D. (2026). *MSE — FOMC Monitor: A multi-market framework for Federal Reserve policy analytics* (v3.0) [Interactive tool]. GitHub. https://github.com/davidbieri/fed-funds

**BibTeX**
```bibtex
@software{bieri2026mse,
  author    = {Bieri, David},
  title     = {{MSE}: A Multi-Market Framework for {Federal Reserve} Policy Analytics},
  year      = {2026},
  version   = {3.0},
  publisher = {GitHub},
  url       = {https://github.com/davidbieri/fed-funds}
}
```

**Chicago**
> Bieri, David. 2026. "MSE: A Multi-Market Framework for Federal Reserve Policy Analytics." v3.0. Interactive tool. GitHub. https://github.com/davidbieri/fed-funds.

---

## References

- Bieri, D., & Chincarini, L. B. (2005). Riding the yield curve: A variety of strategies. *Journal of Fixed Income*, 15(2), 6–35.
- Clarida, R., Galí, J., & Gertler, M. (1999). The science of monetary policy: A New Keynesian perspective. *Journal of Economic Literature*, 37(4), 1661–1707.
- Judd, J. P., & Rudebusch, G. D. (1998). Taylor's rule and the Fed: 1970–1997. *Federal Reserve Bank of San Francisco Economic Review*, 3, 3–16.
- Nelson, C. R., & Siegel, A. F. (1987). Parsimonious modeling of yield curves. *Journal of Business*, 60(4), 473–489.
- Taylor, J. B. (1993). Discretion versus policy rules in practice. *Carnegie-Rochester Conference Series on Public Policy*, 39, 195–214.
- CME Group. (2024). *CME FedWatch methodology*. https://www.cmegroup.com/markets/interest-rates/cme-fedwatch-tool.html

---

## License

MIT © David Bieri, Virginia Tech, School of Public and International Affairs

---

*MSE — after Marriner S. Eccles (1890–1977), principal economic philosopher of the New Deal, architect of the modern Federal Reserve.*

*For educational and research purposes only. Not financial advice.*
