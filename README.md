# Fed Funds Probability Tree

An interactive browser-based tool that implements the **CME FedWatch methodology** for deriving FOMC meeting outcome probabilities from 30-day Federal Funds futures (ZQ), extended with a **Kalshi prediction market comparison** layer.

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)

---

## What it does

The tool replicates the [CME FedWatch calculation](https://www.cmegroup.com/articles/2023/understanding-the-cme-group-fedwatch-tool-methodology.html) from first principles in the browser, covering 12 months forward (currently Apr 2026 – Mar 2027), and overlays Kalshi prediction market data across six analytical panels.

### Six tabs

| Tab | Description |
|-----|-------------|
| **Futures inputs** | Editable table of ZQ prices, FOMC meeting flags, and N/M day splits. All values can be updated manually or refreshed via live web search. |
| **EFFR chain** | Full propagation of the EFFR anchor chain — non-meeting months fix `EFFR(End)ᵀ⁻¹ = EFFR(Avg)ᵀ = EFFR(Start)ᵀ⁺¹`; meeting months are solved via the day-weighted formula. Per-meeting probability bars show the base vs. additional-move split. |
| **Probability tree** | Unconditional cumulative probability lattice across all meetings, rendered as an SVG node graph. Node opacity scales with probability. Summary statistics: P(≥1 cut), P(≥2 cuts), P(≥3 cuts), expected cuts, implied rate. Final rate distribution bar chart. |
| **⚖ Prediction markets** | Nine analytical panels: six comparing CME vs Kalshi vs Polymarket, plus three Polymarket-exclusive panels — see below. |
| **📈 SOFR strip** | 3-month SOFR futures (SR3) analysis — see below. |

### Prediction market panels (Kalshi + Polymarket)

1. **Meeting-by-meeting divergence** — Grouped CME vs Kalshi bars for P(cut) at each FOMC meeting, with a basis pill (Kalshi − CME in pp) highlighting where the two markets disagree.

2. **Cut count distribution** — Side-by-side histogram of P(0/1/2/3/4+ cuts in 2026) from the CME tree vs Kalshi's `kxratecutcount-26dec31` contract, with expected cut count comparison.

3. **Combo path decomposer** — Top 8 most probable hold/cut sequences through the first three meetings, computed from the CME tree and cross-referenced against Kalshi `kxfedcombo` contract prices.

4. **Market-implied dot plot** — Three rate paths on one chart: CME futures chain (blue), Kalshi implied rate levels (orange), and Fed December 2025 SEP median dots (purple). Visual representation of where futures, prediction markets, and the Fed itself disagree about the easing path.

5. **Hike risk monitor** — Semicircular gauge comparing CME-derived max single-meeting hike probability against Kalshi's `kxfedhike` contract, plus a jumbo-cut (>25bp) probability bar from `kxlargecut-26`.

6. **First cut timing** — CME cumulative P(cut has occurred) as a step function across all meetings, with Kalshi's `kxratecut` binary contract prices plotted at their respective horizons (end-2026, end-2027).


7. **Terminal rate distribution** — CME final rate distribution vs Polymarket's discrete end-2026 rate-level market, adjusted for upper/lower bound convention.

8. **Fed Chair nomination** — Polymarket's $230M-volume nominee market (Kevin Warsh hawkish / Rick Rieder dovish) displayed as a hawkishness-tagged bar chart. This is the only panel capturing *policy regime risk* — the probability that the rate function itself changes, not just the level.

9. **Emergency cut gauge** — Polymarket-unique: P(unscheduled FOMC emergency meeting + cut before end-2026). Absent from both CME futures and Kalshi per-meeting markets, which only price scheduled meeting outcomes.

The **Kalshi–Polymarket informational spread** (Poly−Kal basis) is shown alongside each panel — a structural comparison between institutionally-weighted domestic markets (Kalshi, CFTC-regulated) and global retail-weighted markets (Polymarket, crypto-native), revealing where narrative momentum leads fundamentals.

### SOFR strip panels

The 📈 SOFR strip tab covers 3-month SOFR futures (SR3M6–SR3Z7, Jun 2026–Dec 2027) across three panels:

1. **SR3 contract table** — Editable prices for all seven quarterly contracts, with computed implied SOFR rates, ZQ quarterly average EFFR comparison, SOFR–EFFR basis in basis points, and instantaneous 3M forward rates derived from adjacent contracts.

2. **Three-curve forward rate chart** — SR3 implied SOFR (green, solid), ZQ quarterly average EFFR (blue), instantaneous 3M forward rates (green dashed), and Fed December 2025 SEP median dots (purple). The forward curve trough marks the quarter where markets price peak easing — distinct from the terminal rate level implied by the strip's endpoint.

3. **Basis bars + strip statistics** — Per-quarter SOFR–EFFR basis bars and a summary panel: near-term basis, SOFR terminal rate (Dec 2027), EFFR terminal rate (Mar 2027), strip slope, and the instantaneous forward trough quarter.

---

## CME FedWatch methodology

The tool implements the full methodology documented by CME Group:

1. **Anchor non-meeting months**: `EFFR(End)ᵀ⁻¹ = EFFR(Avg)ᵀ = EFFR(Start)ᵀ⁺¹`
2. **Solve meeting months** via day-weighted formula:
   ```
   EFFR(Start) = { EFFR(Avg) − (M/N+M) · EFFR(End) } / (N/N+M)
   ```
   where N = days from month start through meeting day, M = remaining days.
3. **Expected EFFR change** = EFFR(End) − EFFR(Start)
4. **Convert to 25bp moves**: divide by 0.25
5. **Split into characteristic + mantissa**: e.g. 2.9 moves → 2 (char) + 0.9 (mantissa)
6. **Probabilities**: P(base move) = 1 − mantissa; P(larger move) = mantissa
7. **Compound unconditionally** across meetings to build the full lattice

---

## Live data refresh

The tool uses the **Anthropic API** (claude-sonnet with web search) to pull current ZQ futures prices and Kalshi market data on demand. Two independent refresh buttons:

- **Refresh ZQ** — fetches current CME 30-day Fed Funds futures settlement prices and updates the FOMC meeting schedule
- **Fetch Kalshi data** — fetches per-meeting decision probabilities, cut count distribution, hike risk, and binary cut-by-date contracts from Kalshi

### API key

**Inside claude.ai**: the key is injected automatically — no action needed.

**Standalone (local browser)**: paste your Anthropic API key (`sk-ant-...`) into the field in the top-right corner. Get one at [console.anthropic.com](https://console.anthropic.com). Each refresh call costs well under $0.01.

---

## GitHub Pages

The tool is live at **[davidbieri.github.io/fed-funds](https://davidbieri.github.io/fed-funds)** — no login, no server, opens directly in any browser. The `index.html` at the repo root redirects to the main file.

To enable Pages on a fork: **Settings → Pages → Source: Deploy from branch → Branch: main / (root) → Save**.

## Usage

No build step, no dependencies. Open `fed_funds_probability_tree.html` directly in any modern browser.

```bash
git clone https://github.com/davidbieri/fed-funds.git
cd fed-funds
open fed_funds_probability_tree.html   # macOS
# or: xdg-open fed_funds_probability_tree.html  (Linux)
# or: start fed_funds_probability_tree.html      (Windows)
```

All values in the **Futures inputs** tab are editable at any time — you can override any live-fetched price manually, adjust N/M day splits, or toggle which months contain FOMC meetings.

---

## Data sources

| Source | Usage |
|--------|-------|
| CME 30-Day Fed Funds (ZQ) futures | Anchor prices for the EFFR chain |
| Federal Reserve Bank of New York | EFFR daily publication |
| Kalshi | Per-meeting decision markets, cut count, hike risk, rate level |
| CME 3-Month SOFR (SR3) futures | Forward rate curve, basis analysis, instantaneous forwards |
| Federal Reserve SEP (Dec 2025) | Dot plot overlaid on Panel ④ |

---

## Limitations and caveats

- Probabilities assume rate changes are uniformly sized in **25bp increments** and that EFFR reacts proportionally.
- The EFFR is bounded below at 0% by convention (though this is not binding in the current rate environment).
- Kalshi data fetched via web search may reflect slightly stale settlement prices depending on market hours.
- This tool is for **research and educational purposes only** and does not constitute investment advice.

---

## Background

Developed as part of a broader research program connecting financial market microstructure, monetary policy, and geoeconomic analysis. The CME FedWatch methodology was reverse-engineered from the [official CME documentation](https://www.cmegroup.com/articles/2023/understanding-the-cme-group-fedwatch-tool-methodology.html); the Kalshi comparison layer is an original extension.

---

## License

MIT — see [LICENSE](LICENSE).

---

*Last updated: March 2026 · v3.0*
