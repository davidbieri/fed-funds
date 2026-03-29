# MSE: A Multi-Market Framework for Federal Reserve Policy Analytics

**A multi-market framework for Federal Reserve policy analytics.**

[![Version](https://img.shields.io/badge/version-4.0-blue)](https://github.com/davidbieri/fed-funds/blob/main/CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub Pages](https://img.shields.io/badge/deployed-GitHub%20Pages-brightgreen)](https://davidbieri.github.io/fed-funds/)

**Live tool:** [davidbieri.github.io/fed-funds](https://davidbieri.github.io/fed-funds/)  
**Virginia Tech · School of Public and International Affairs**

> *MSE — after Marriner S. Eccles (1890–1977), principal economic philosopher of the New Deal, architect of the modern Federal Reserve.*

> For educational and research purposes only. Not financial advice.

---

## Overview

MSE implements the CME FedWatch methodology from first principles — deriving per-meeting cut/hold/hike probabilities from 30-day Fed Funds (ZQ) futures prices — and extends it with five layers of analytical context: prediction market comparison (Kalshi and Polymarket), SOFR strip basis analysis, Nelson-Siegel term structure modeling, Taylor Rule prescriptive analysis, a complete Bieri-Chincarini (2005) riding-the-yield-curve calculator, and — in v4.0 — a Policy Confidence Index synthesizing ZQ signal strength, SR3 transmission quality, Atlanta Fed MPT options conviction, and crowd sentiment alignment, plus a Financial Stability tab grounded in the BIS/Mehrling financial cycle framework.

All ten tabs share a single state machine. Changing any upstream input — a ZQ price, a macro variable, a scenario preset — propagates instantly through the entire `useFinance` memoization pipeline to every downstream tab.

No build step. No npm. Open directly in a browser or deploy as a static file.

---

## Ten tabs

| # | Tab | Description |
|---|-----|-------------|
| 01 | **Futures Inputs** | Editable ZQ price table (Apr 2026–Mar 2027), FOMC meeting flags, N/M day splits. Source of truth for all tabs. |
| 02 | **EFFR Chain** | EFFR anchor propagation: non-meeting months fix EFFR(End) = EFFR(Avg) = EFFR(Start+1); meeting months solved via the day-weighted FedWatch formula. |
| 03 | **Probability Tree** | SVG unconditional probability lattice across all FOMC meetings. Node opacity ∝ probability. Summary stats: P(≥1/2/3 cuts), expected cuts, implied rate, final distribution. |
| 04 | **Prediction Markets** | Ten-panel comparison: CME vs Kalshi vs Polymarket across nine panels + panel ⑩ Signal/Noise Decomposition (PCI breakdown, platform divergence, conviction spread). |
| 05 | **SOFR Strip** | 7 quarterly SR3 contracts (SR3M6–SR3Z7, Jun 2026–Dec 2027). Three-curve chart: SOFR strip, ZQ quarterly EFFR avg, instantaneous 3M forward. Per-quarter SOFR–EFFR basis in bp. |
| 06 | **Dashboard** | Unified cross-market view: stat cards including PCI score + regime dial + sentiment gap sparklines, combined rate path chart, divergence table, financial stability row (NFCI, leaning premium, cycle phase). |
| 07 | **Yield Curve** | Nelson-Siegel fitting over 7 Treasury CMT maturities (3mo–30yr) via 2,000-iteration Nelder-Mead. Log-scale spot and instantaneous forward curves. Scenario-shocked paths per FOMC meeting using Bieri-Chincarini empirical factor loadings. |
| 08 | **Taylor Rule** | Five rule variants: Taylor 1993, Balanced/Yellen 2012, Yellen Alt1 (Okun's Law), Clarida-Gali-Gertler inertial (ρ=0.85), Judd-Rudebusch ECM (Bieri-Chincarini 2005, Eqs. 23–25). Prescribed vs CME path chart, Taylor−CME gap, per-meeting rule matrix. |
| 09 | **Riding** | Bieri-Chincarini (2005) riding-the-yield-curve calculator: 7×5 HPR matrix (6mo–30yr instruments × 1mo–18mo horizons), excess HPR and break-even cushion in bp, five-filter cascade, duration-neutral barbell decomposition. |
| 10 | **Balance Sheet: QE · QT Monitor** | 4-panel layout: composition stacked bars with KVJ (2011) shadow rate decomposition, monthly runoff pace vs cap (Treasury + MBS), corridor mechanics (IORB/EFFR/SVR/RRP), two-instrument policy space scatter with 9 historical episodes. |
| 11 | **Financial Stability** | BIS/Mehrling financial cycle framework: NFCI vs EFFR path chart, finance-neutral Taylor Rule (λ_fc slider), global dollar stress gauge (swap lines + basis), doom loop scatter (term premium vs HY OAS), five-axis financial cycle phase clock. |

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

NS factor interpretation: β₀ = level (long-run rate); β₁ = slope (short-end loading, β₀ + β₁ = instantaneous short rate); β₂ = curvature (medium-maturity hump); λ = decay constant; term premium ≈ β₀ − EFFR.

### Taylor Rule suite (tab 08)

**Atlanta Fed generalized form (base for variants 1–3):**

$$\hat{i}_t = \rho \cdot i_{t-1} + (1-\rho)\left[r^* + \pi^* + 1.5(\pi_t - \pi^*) + \beta \cdot \text{gap}_t\right]$$

where gap = output gap (% of potential GDP) or −2×(u − u\*) via Okun's Law.

**Judd-Rudebusch ECM — designated signal source (Bieri-Chincarini 2005, Eqs. 23–25):**

$$i^*_t = r^* + \pi^* + \lambda_1(\pi_t - \pi^*) + \lambda_2\,\text{gap}_t + \lambda_3\,\text{gap}_{t-1}$$

$$\Delta i_t = \gamma(i^*_t - i_{t-1})$$

Partial adjustment toward the long-run equilibrium target. Highest R² for FFTR prediction among all variants tested (Bieri-Chincarini, Exhibit 7). The JR ECM signal feeds the Riding Calculator filter cascade and the Dashboard regime summary.

| Variant | Specification | Key parameters |
|---------|--------------|----------------|
| Taylor 1993 | Original specification | α=1.5, β=0.5, GDP output gap |
| Balanced / Yellen 2012 | "Balanced approach" rule | α=β=1.0 |
| Yellen Alt1 | Unemployment gap via Okun's Law | gap = −2×(u−u\*), FOMC SEP r\* |
| Clarida-Gali-Gertler inertial | High-persistence smoothing | ρ=0.85 |
| Judd-Rudebusch ECM ★ | Dynamic partial adjustment | λ₁, λ₂, λ₃, γ — editable |

### Policy Confidence Index (tabs 04, 06)

A four-component composite index (0–100) synthesizing signals across instrument types:

$$\text{PCI} = w_1 \cdot f(\Delta\text{ZQ}) + w_2 \cdot f(\text{basis}^{-1}) + w_3 \cdot f(\sigma^{-1}_{\text{options}}) + w_4 \cdot f(\text{SentimentGap}^{-1})$$

| Component | Source | Formula | Default weight |
|---|---|---|---|
| Signal strength | ZQ strip slope | `clamp(0.5 − slope_bp/200, 0, 1)` | 0.35 |
| Transmission quality | SR3 basis | `clamp(1 − max(0, basis_bp−5)/30, 0, 1)` | 0.25 |
| Options conviction | Atlanta Fed MPT IQR | `clamp(1 − avgIQR/100, 0, 1)` | 0.25 |
| Sentiment alignment | Crowd vs ZQ gap | `clamp(1 − avgAbsGap/0.25, 0, 1)` | 0.15 |

Four regime labels: **Uncertain** (0–25) · **Divergent** (25–50) · **Forming** (50–75) · **Convicted** (75–100). Weights are user-editable.

### Policy Confidence Index — academic framing

The PCI's four components map onto a well-defined heterogeneous-markets literature, each component formalizing a distinct theoretical mechanism:

**Signal strength and transmission (ZQ + SR3)** — The ZQ chain and SR3 strip represent the institutional consensus, where capital is at risk and prices are arbitraged continuously. These are the "smart money" layers in the sense of Grossman & Stiglitz (1980): prices aggregate dispersed information efficiently under competitive conditions. The SOFR–EFFR basis adds a microstructure dimension — transmission quality is not a given but varies with reserve scarcity, collateral availability, and repo market functioning.

**Options conviction (Atlanta Fed MPT)** — The information content of options prices over and above futures prices is the subject of Jackwerth & Rubinstein (1996). Options reveal the full shape of the implied distribution — variance, skewness, tail probabilities — that futures prices, being expectations, cannot. The IQR of the MPT distribution is a direct measure of how much uncertainty the volatility market is pricing around the futures-implied central forecast. A narrow IQR signals that sophisticated hedgers are paying little for tail protection; a wide IQR signals genuine distributional uncertainty even when the mean is stable. The ConvictionSpread formalizes this comparison between options-implied conviction and crowd consensus.

**Sentiment alignment (Kalshi / Polymarket vs ZQ)** — The SentimentGap — the spread between crowd-aggregated prediction market probabilities and ZQ-derived probabilities — maps onto two complementary theoretical frameworks. Wolfers & Zitzewitz (2004) establish that prediction markets are efficient information aggregators under appropriate conditions, but are susceptible to narrative cascades when a large fraction of participants trade on sentiment rather than fundamentals. The persistence of the SentimentGap when it does not mean-revert is consistent with Shleifer & Vishny (1997): institutional arbitrageurs who recognize the mispricing cannot fully close it because of noise-trader risk and capital constraints, so the gap can persist beyond its "fundamental" horizon.

**Platform divergence (Polymarket vs Kalshi)** — The PlatformDiv — the spread between Polymarket and Kalshi per-meeting probabilities — operationalizes Kurz's (1994) rational beliefs framework. Agents with identical information sets can rationally hold heterogeneous beliefs when their models of the world differ. Kalshi's regulated, domestic, institutionally-weighted participant base and Polymarket's global, retail, crypto-native base constitute two distinct agent types with structurally different information processing and loss functions. When PlatformDiv is large and persistent, it is not merely noise — it reflects genuine disagreement between agent types about the probability model governing future Fed decisions, exactly the condition Kurz's framework predicts under model uncertainty.

**The novel synthesis** — No existing public tool reads all four layers simultaneously and produces a conviction-adjusted policy signal. The PCI is constructed on the premise that the joint distribution of (futures slope, basis quality, options IQR, sentiment gap) contains more information about the true state of policy expectations than any single source alone — a direct application of the information aggregation principle. The weights are calibrated heuristically and made user-editable precisely because the relative informativeness of each layer varies across the business cycle: in crisis episodes, options conviction and swap line stress dominate; in normal regimes, the ZQ signal and sentiment alignment carry more weight.

### Finance-neutral natural rate (tab 10)

$$r^*_{\text{finance}} = r^*_{\text{standard}} + \lambda_{fc} \times \text{NFCI} \times 100\text{bp}$$

The leaning premium (λ_fc × NFCI × 100bp) measures how much additional tightening financial stability considerations warrant beyond the standard inflation-neutral rule. Positive NFCI = tight conditions = higher finance-neutral r\*. Based on Borio, Disyatat & Juselius (2016) finance-neutral natural rate framework.

### Riding the yield curve (tab 09)

Zero-coupon excess holding-period return (Bieri-Chincarini 2005, Eq. 12):

$$\text{XHPR}(m, h) = \text{HPR}(m, h) - \text{mmHPR}(h)$$

Break-even cushion (Eqs. 15–16):

$$\text{Cushion}(m,h) = \frac{y(m) - y(h)}{D(m,h)} \times 100 \text{ bp}$$

Duration-neutral barbell weight ω (Eq. 29) and riding HPR decomposition (Eq. 30) are computed in the barbell panel. The aggregate riding signal feeds the Dashboard stat card. Five configurable filters gate the signal cascade: positive slope, positive cushion, 75th-percentile cushion, Taylor (JR ECM) signal, and CME expectations signal.

---

## Data sources

| Source | Data | Refresh |
|--------|------|---------|
| **CME** | ZQ 30-day Fed Funds futures (Apr 2026–Mar 2027) | Refresh ZQ button |
| **CME** | SR3 3-month SOFR futures (SR3M6–SR3Z7) | Fetch SR3 button |
| **Kalshi** | Per-meeting decision markets, cut count, hike risk, jumbo cut, implied rates | Fetch Kalshi button |
| **Polymarket** | Per-meeting probs, cut count (13 outcomes), terminal rate, Fed Chair nomination ($230M vol), emergency cut, first-cut timing | Fetch Polymarket button |
| **Federal Reserve** | EFFR (NY Fed daily), SOFR fixing, Dec 2025 SEP dot plot, H.15 CMT rates (7 tenors) | H.15 fetch via Yield Curve tab |
| **Atlanta Fed MPT** | Options-implied SOFR distribution (p25/p50/p75/IQR) for 4 quarterly SR3 contracts | Fetch MPT button |
| **FRED** | NFCI (Chicago Fed, weekly), CB liquidity swap outstandings (SWPT), ICE BofA HY OAS (BAMLH0A0HYM2) | Fetch FRED button |

Live refresh uses the Anthropic API with web search (`claude-sonnet-4-6`). Inside claude.ai the API key is injected automatically. For standalone use, paste an `sk-ant-...` key into the field in the top-right corner of the tool.

---

## Prediction market panels (tab 04)

Nine panels comparing CME-derived probabilities against Kalshi and Polymarket, plus a tenth Signal/Noise panel:

1. **Meeting divergence** — 3 bars per meeting (CME / Kalshi / Polymarket) with Kal−CME and Poly−Kal basis pills
2. **Cut count distribution** — P(0–5+ cuts in 2026) across all three sources (Polymarket has 13 discrete strikes)
3. **Combo path decomposer** — Top 8 hold/cut path sequences vs Kalshi kxfedcombo contracts
4. **Market-implied dot plot** — CME chain + Kalshi implied rates + Fed Dec 2025 SEP on one chart
5. **Hike risk monitor** — Semicircular gauge with three concentric arcs (CME / Kalshi / Polymarket)
6. **First-cut timing** — CME step curve + Kalshi year-end circles + Polymarket per-meeting diamonds
7. **Terminal rate distribution** — CME final distribution vs Polymarket end-2026 market
8. **Fed Chair regime risk** — Polymarket $230M nominee market with hawkishness tags
9. **Emergency cut gauge** — P(unscheduled FOMC cut before end-2026), Polymarket-unique
10. **Signal/Noise decomposition** — PCI component breakdown, platform divergence, conviction spread (Atlanta Fed MPT IQR vs crowd SentimentGap)

The key analytical signal is the *basis* between markets — Kal−CME captures the regulated vs futures spread; Poly−Kal captures the information gap between institutional/domestic (Kalshi) and global/retail/crypto-native (Polymarket) sentiment.

---

## Historical scenario presets

Ten named ZQ price configurations grouped by historical era. Activating a scenario reprices the entire chain — probability tree, Kalshi/Polymarket overlays, yield curve scenario fan, Taylor prescriptions, and riding HPR matrix — instantly. A second click on the active scenario restores the previous prices. An amber compare overlay applies a second scenario alongside the active one for parallel analysis.

ZQ prices are derived from the CME N/M weighting formula at each FOMC meeting. Starting EFFR: 3.64% (lower bound 3.50%). Meeting N/M splits: Apr(28,2), Jun(17,13), Jul(28,3), Sep(16,14), Oct(28,3), Dec(9,22), Jan(27,4), Mar(17,14).

### Easing scenarios

| Scenario | Total | End rate | Apr | Jun | Jul | Sep | Oct | Dec | Jan | Mar |
|----------|------:|--------:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Soft Landing '95** | −25bp | 3.39% | hold | −25 | hold | hold | hold | hold | hold | hold |
| **Insurance '98/'19** | −75bp | 2.89% | hold | −25 | −25 | −25 | hold | hold | hold | hold |
| **Measured Pace** | −100bp | 2.64% | hold | −25 | hold | −25 | hold | −25 | hold | −25 |
| **Recession '90–91** | −150bp | 2.14% | −25 | −25 | −25 | −25 | −25 | −25 | hold | hold |
| **Dot-com '01** | −225bp | 1.39% | hold | −50 | −50 | −50 | −25 | −25 | −25 | hold |

**Soft Landing '95** — One 25bp cut at Jun, then a prolonged hold. After the 1994 hiking campaign, Greenspan delivered three 25bp cuts in mid-1995 as insurance; growth never faltered. The most benign plausible outcome.

**Insurance '98/'19** — Three consecutive 25bp cuts (Jun, Jul, Sep), then hold. Mirrors the Sep–Nov 1998 response to LTCM and Russia default, and the 2019 mid-cycle adjustment (Jul–Oct). Easing that stops once tail risk subsides.

**Measured Pace** — One 25bp cut per quarter (Jun, Sep, Dec, Mar). Mirror image of the 2015–18 liftoff cadence — methodical, fully telegraphed, consistent with a last-mile disinflation slowdown that keeps growth intact.

**Recession '90–91** — 25bp cut at every meeting Apr–Dec (six meetings), then pause. Mirrors the 1989–91 easing cycle during the S&L crisis and Gulf War recession. Persistent incremental pressure; economy tips into recession but the financial system stays intact.

**Dot-com '01** — Hold in Apr, then three 50bp cuts (Jun, Jul, Sep) and three 25bp cuts (Oct, Dec, Jan). Mirrors the Fed's 2001 response to the dot-com collapse and 9/11. Severe demand shock with no systemic financial seizure.

### Crisis scenarios

| Scenario | Total | End rate | Apr | Jun | Jul | Sep | Oct | Dec | Jan | Mar |
|----------|------:|--------:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **GFC '08** | −300bp | 0.64% | −25 | −50 | −75 | −75 | −50 | −25 | hold | hold |
| **Volcker '82** | −350bp | 0.14% | −50 | −75 | −75 | −75 | −50 | −25 | hold | hold |

**GFC '08** — Crisis-level easing driving EFFR to ~0.64% by year-end. The Fed cut 500bp in 15 months during the GFC. This analog applied from today's starting level is the most severe single-year easing in modern central banking history.

**Volcker '82** — Maximum historical easing velocity: 50bp (Apr), then four jumbo cuts, reaching near-zero. A purely illustrative extreme for stress-testing the probability tree at the zero lower bound.

### Tightening scenarios

| Scenario | Total | End rate | Apr | Jun | Jul | Sep | Oct | Dec | Jan | Mar |
|----------|------:|--------:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Stagflation '70s** | +100bp | 4.64% | hold | hold | +25 | +25 | hold | +25 | hold | +25 |
| **Re-norm '22** | +350bp | 7.14% | +50 | +75 | +75 | +75 | +50 | +25 | hold | hold |

**Stagflation '70s** — Hold through Jun, then gradual re-tightening: 25bp hikes at Jul, Sep, Dec, Mar. The defining failure of the Burns/Miller era — premature loosening followed by stubborn inflation forcing re-tightening. The scenario where the Fed loses the narrative on disinflation.

**Re-norm '22** — A carbon-copy of the 2022 hiking cycle applied from today's starting rate: 50bp (Apr), 75bp (Jun, Jul, Sep), 50bp (Oct), 25bp (Dec) — total +350bp driving EFFR to ~7.14%. The scenario where inflation re-accelerates sharply or the new Fed Chair abandons the easing path entirely. Useful for Kalshi/Polymarket hike-risk calibration.

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

**State**: `useReducer` with a 21-action state machine. All derived values computed in a `useFinance` hook (pure memoized pipeline). Ten tabs render from shared state — no globals.

**Signal chain**: ZQ prices → EFFR chain → Probability tree → Prediction markets overlay → SOFR strip → Taylor Rule (JR ECM signal) → Yield Curve (NS + scenario shocks) → Riding (HPR + filter cascade) → Dashboard (unified view + PCI) → Financial Stability (BIS cycle framework).

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
- PCI conviction component requires the Atlanta Fed MPT live fetch; seed IQR values of 75bp are used until refreshed — treat seed-based PCI scores as approximate.
- Financial Stability panels require the FRED live fetch for current NFCI, swap line, and HY spread readings; seed values reflect approximate conditions at time of release.
- This tool is for **research and educational purposes only** and does not constitute investment advice.

---

## Background

Developed as part of a broader research program connecting financial market microstructure, monetary policy, and geoeconomic analysis at Virginia Tech's School of Public and International Affairs. The CME FedWatch methodology was reverse-engineered from the [official CME documentation](https://www.cmegroup.com/articles/2023/understanding-the-cme-group-fedwatch-tool-methodology.html); the Kalshi and Polymarket comparison layers, the Nelson-Siegel term structure module, the Taylor Rule suite, and the riding-the-yield-curve calculator are original extensions grounded in the academic literature cited below.

**Relationship to Bloomberg WIRP**

MSE's core probability methodology (tabs 01–03) is the open-source equivalent of Bloomberg's WIRP function in Futures Mode (*World Interest Rate Probability*, `WIRP <GO>`). WIRP Futures Mode derives the percentage of a 25bp hike or cut priced in at each FOMC meeting from the same ZQ Fed Funds futures contracts using the same N/M day-weighting procedure. MSE independently replicates this methodology from CME's published documentation; its ZQ-derived probabilities are verified to be consistent with WIRP Futures Mode output, providing an external methodological benchmark.

Where MSE extends beyond WIRP: prediction market integration (Kalshi, Polymarket), SOFR strip basis analysis, Nelson-Siegel term structure fitting, Taylor Rule prescriptive comparison, Bieri-Chincarini riding analytics, and — in v4.0 — a Policy Confidence Index synthesizing options-implied conviction (Atlanta Fed MPT), sentiment alignment, and financial conditions. WIRP provides none of these layers. MSE's pedagogical and research value lies in making the Bloomberg-equivalent core methodology fully transparent and open, then extending it into territory that no single commercial terminal covers.

---

## Citation

**APA**
> Bieri, D. (2026). *MSE: A Multi-Market Framework for Federal Reserve Policy Analytics* (v4.0) [Interactive tool]. Virginia Tech School of Public and International Affairs. GitHub. https://github.com/davidbieri/fed-funds

**BibTeX**
```bibtex
@software{bieri2026mse,
  author      = {Bieri, David},
  title       = {{MSE}: A Multi-Market Framework for {Federal Reserve} Policy Analytics},
  year        = {2026},
  version     = {4.0},
  institution = {Virginia Tech School of Public and International Affairs},
  publisher   = {GitHub},
  url         = {https://github.com/davidbieri/fed-funds}
}
```

**Chicago**
> Bieri, David. 2026. "MSE: A Multi-Market Framework for Federal Reserve Policy Analytics." v4.0. Interactive tool. Virginia Tech School of Public and International Affairs. GitHub. https://github.com/davidbieri/fed-funds.

---

## References

- Bieri, D., & Chincarini, L. B. (2005). Riding the yield curve: A variety of strategies. *Journal of Fixed Income*, 15(2), 6–35.
- Borio, C., Disyatat, P., & Juselius, M. (2016). Rethinking potential output: Embedding information about the financial cycle. *Oxford Economic Papers*, 69(3), 655–677.
- Clarida, R., Galí, J., & Gertler, M. (1999). The science of monetary policy: A New Keynesian perspective. *Journal of Economic Literature*, 37(4), 1661–1707.
- Grossman, S. J., & Stiglitz, J. E. (1980). On the impossibility of informationally efficient markets. *American Economic Review*, 70(3), 393–408.
- Jackwerth, J. C., & Rubinstein, M. (1996). Recovering probability distributions from option prices. *Journal of Finance*, 51(5), 1611–1631.
- Judd, J. P., & Rudebusch, G. D. (1998). Taylor's rule and the Fed: 1970–1997. *Federal Reserve Bank of San Francisco Economic Review*, 3, 3–16.
- Kurz, M. (1994). On the structure and diversity of rational beliefs. *Economic Theory*, 4(6), 877–900.
- Mehrling, P. (2016, June 28). BIS looks through the financial cycle. *Boston University Global Development Policy Center Blog*. https://sites.bu.edu/perry/2016/06/28/bis-looks-through-the-financial-cycle/
- Nelson, C. R., & Siegel, A. F. (1987). Parsimonious modeling of yield curves. *Journal of Business*, 60(4), 473–489.
- Shleifer, A., & Vishny, R. W. (1997). The limits of arbitrage. *Journal of Finance*, 52(1), 35–55.
- Taylor, J. B. (1993). Discretion versus policy rules in practice. *Carnegie-Rochester Conference Series on Public Policy*, 39, 195–214.
- Wolfers, J., & Zitzewitz, E. (2004). Prediction markets. *Journal of Economic Perspectives*, 18(2), 107–126.
- CME Group. (2024). *CME FedWatch methodology*. https://www.cmegroup.com/markets/interest-rates/cme-fedwatch-tool.html
- Federal Reserve Bank of Atlanta. (2026). *Market Probability Tracker*. https://www.atlantafed.org/research-and-data/data/market-probability-tracker

---

## License

MIT © David Bieri, Virginia Tech School of Public and International Affairs

---

*MSE — after Marriner S. Eccles (1890–1977), principal economic philosopher of the New Deal, architect of the modern Federal Reserve.*

*For educational and research purposes only. Not financial advice.*
