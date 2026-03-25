# Fed Funds Probability Tree — Claude Code Context

## What this project is

A standalone browser-based tool implementing the CME FedWatch methodology for deriving
FOMC meeting outcome probabilities from 30-day Federal Funds (ZQ) futures, with Kalshi
and Polymarket prediction market overlays and a SOFR futures strip. 12 months forward.
No build step, no dependencies — single HTML file plus supporting docs.

## Repo structure

```
fed-funds/
├── fed_funds_probability_tree.html   # Main app — ~1,290 lines, everything inlined
├── index.html                        # GitHub Pages redirect to the main file
├── README.md                         # Full methodology + usage documentation
└── CHANGELOG.md                      # v1.0 / v2.0 / v3.0 history
```

GitHub Pages live URL: davidbieri.github.io/fed-funds

## Architecture of the main HTML file

Single file: inlined CSS + vanilla JS + SVG rendering. No framework, no bundler.

**Five tabs** — each rendered by a dedicated function:
- `fwRenderInputs()` — editable ZQ futures price table
- `fwRenderChain()` — EFFR chain propagation (anchor months → meeting months)
- `fwRenderTree()` — unconditional probability lattice (SVG node graph)
- `fwRenderKalshi()` — prediction markets tab: 9 panels (CME / Kalshi / Polymarket)
- `fwRenderSOFR()` — SR3 SOFR strip: 3-curve forward chart + basis analysis

**Core calculation chain** (pure functions, called on every input change):
```
fwChain(fwD) → fwMeetings(fwD, chain) → fwDist(meetings)
```

**Global state** (mutable module-level objects):
- `fwD[]` — 12 ZQ contract rows (Apr 2026 – Mar 2027)
- `kalshiD{}` — Kalshi seed data + fetched state
- `polyD{}` — Polymarket seed data + fetched state
- `srD[]` — 7 SR3 SOFR quarterly contract rows

**Live data refresh** — four independent async functions, each calls Anthropic API
with web_search tool, parses JSON response, merges into global state, re-renders:
- `refreshData()` — ZQ prices + FOMC schedule
- `fetchKalshi()` — Kalshi decision/cut-count/hike/rate markets
- `fetchPolymarket()` — Polymarket per-meeting/terminal/chair/emergency markets
- `fetchSOFR()` — SR3 settlement prices

**SVG rendering** — all panels use string-concatenation SVG inside JS functions.
The probability tree lattice is hand-computed (node positions, edge widths, opacity).
Bar charts, gauges, and step functions are also manual SVG — no charting library.

**CSS** — CSS custom properties for theming (`--color-*`, `--c-cme`, `--c-kal`,
`--c-poly`, `--c-sofr`). Dark mode via `prefers-color-scheme`. No external stylesheet.

## Color system

| Variable     | Color            | Used for                   |
|--------------|------------------|----------------------------|
| `--c-cme`    | Blue #3b82f6     | CME futures data           |
| `--c-kal`    | Orange #f97316   | Kalshi markets             |
| `--c-poly`   | Cyan #06b6d4     | Polymarket                 |
| `--c-dots`   | Purple #8b5cf6   | Fed SEP dot plot           |
| `--c-sofr`   | Green #10b981    | SR3 SOFR strip             |
| `--c-sofr-fwd` | Lt green #34d399 | Instantaneous forward rates |

## Working preferences

**Edits:** always use targeted `str_replace` over full rewrites. The file is ~1,290
lines — partial edits preserve working state and minimize regression risk.

**Syntax verification:** after any JS change, extract the `<script>` block and run
`node --check` on it. Brace depth must end at 0. Backtick count must be even.

**Orphaned code:** watch for stale code fragments left between functions after
`str_replace` operations — they cause subtle parse errors. Verify with brace counting
after each edit session.

**Non-ASCII in JS comments:** box-drawing characters (─, ═) in section dividers cause
`node --check` false positives. Use plain ASCII (---, ===) in JS comments.

**Git:** short-lived GitHub personal access tokens (classic, `repo` scope only),
revoked immediately after each push. Commit messages should be descriptive — see
existing commits for style reference.

**Template literals in prompts:** the Anthropic API prompt strings inside
`fetchKalshi()` and `fetchPolymarket()` span multiple lines inside backtick template
literals and contain JSON with curly braces. Do not accidentally close these early.

**onchange not oninput:** input fields use `onchange` (fires on blur/Enter) rather than
`oninput` (fires on every keystroke) to avoid re-render loops mid-typing.

## FOMC schedule (current seed data)

| Month    | Meeting      | N  | M  |
|----------|--------------|----|----|
| Apr 2026 | Apr 28–29 ●  | 28 | 2  |
| May 2026 | (anchor)     | —  | —  |
| Jun 2026 | Jun 17–18 ●  | 17 | 13 |
| Jul 2026 | Jul 28–29 ●  | 28 | 3  |
| Aug 2026 | (anchor)     | —  | —  |
| Sep 2026 | Sep 16–17 ●  | 16 | 14 |
| Oct 2026 | Oct 28–29 ●  | 28 | 3  |
| Nov 2026 | (anchor)     | —  | —  |
| Dec 2026 | Dec 9–10 ●   | 9  | 22 |
| Jan 2027 | Jan 27–28 ●  | 27 | 4  |
| Feb 2027 | (anchor)     | —  | —  |
| Mar 2027 | Mar 17–18 ●  | 17 | 14 |

N = calendar days from month start through and including meeting day.
M = remaining days after. N + M = total days in month.

## Planned improvements (backlog)

- **Landing page upgrade** — replace bare `index.html` redirect with a proper landing
  page: description, citation block (APA/BibTeX/Chicago), version badge, launch button.
  Pattern to follow: `davidbieri/oil-scenarios` oil-calculator/index.html.
- **Scenario toggle system** — named FOMC scenarios (e.g., "Warsh hawkish pivot",
  "emergency cut", "stagflation hold") that pre-configure ZQ seed prices and
  prediction market data. Modelled on the 12-scenario system in oil-scenarios.
- **Parameter inference from live data** — on ZQ refresh, compute the strip slope and
  surface the implied rate regime as a contextual annotation on the inputs tab.
- **React migration (long-term)** — calculation engine functions (`fwChain`,
  `fwMeetings`, `fwDist`) are pure and can be lifted unchanged into a React app.
  The non-lattice visualizations (bar charts, basis plots, SOFR curves) would benefit
  most from declarative charting (Recharts or D3).

## Related repo

`davidbieri/oil-scenarios` — Monte Carlo oil & gasoline price calculator.
React JSX architecture, 8 stochastic models, 12 historical scenarios, live Yahoo
Finance fetch (WTI/OVX/VIX/curve slope), PADD regional gasoline pricing.
Has an `oil-calculator/` HTML wrapper landing page — the deployment pattern to
replicate here for the fed-funds GitHub Pages landing page.
