# MSE: A Multi-Market Framework for Federal Reserve Policy Analytics

## What this is
Single-file HTML tool implementing CME FedWatch methodology (30-day ZQ futures chain → unconditional probability lattice) with Kalshi + Polymarket prediction market comparison, SOFR strip, Dashboard, Nelson-Siegel yield curve, Taylor Rule suite (five variants incl. Judd-Rudebusch ECM), and Bieri-Chincarini (2005) riding-the-yield-curve calculator. Deployed via GitHub Pages at davidbieri.github.io/fed-funds.

## File structure
```
fed_funds_probability_tree.html  ← Main tool (~1,290 lines, all-in-one)
index.html                       ← Landing page (launches tool in iframe modal)
README.md
CHANGELOG.md
```

## Architecture
- **React 18 CDN, zero dependencies** — no build step, no bundler, open directly in browser
- **State**: `useReducer` with 17-action state machine; all derived values in `useFinance` memoized hook
- **Rendering**: SVG string concatenation + React JSX components; 9 tabs render from shared state
- **Live data**: Anthropic API + web search (claude-sonnet-4-20250514), separate refresh buttons per data source
- **API key**: injected automatically inside claude.ai; manual sk-ant-... field for standalone use
- **Signal chain**: ZQ prices → EFFR chain → Prob. tree → Pred. markets → SOFR strip → Dashboard → Yield Curve → Taylor Rule (JR ECM signal) → Riding Calculator

## Editing conventions
- **Prefer targeted str_replace over full rewrites** — file is large (~3,500 lines), full rewrites risk regression
- **Syntax check after every JS change**: `node --check fed_funds_v3.html`
- **Non-ASCII box-drawing chars in JS comments cause node --check false positives** — use plain ASCII `---` not box chars
- **Input handlers**: always `onchange`, never `oninput` — prevents re-render loops
- **Tab IDs**: inputs, chain, tree, kalshi, sofr, dashboard, taylor, yieldcurve, riding — update SET_TAB reducer if adding tabs

## Color tokens
--c-cme blue, --c-kal orange, --c-poly cyan, --c-dots purple, --c-sofr green, --c-sofr-fwd light green
--c-taylor rose (#e11d48), --c-overlay amber (#f59e0b), --c-riding violet (implicit #8b5cf6)

## FOMC schedule (seed, corrected)
Apr 28-29 (Powell's last), Jun 17-18, Jul 28-29, Sep 16-17, Oct 28-29, Dec 9-10, Jan 27-28, Mar 17-18
Anchor months (no meeting): May, Aug, Nov, Feb

## Git workflow
- Short-lived personal access tokens (classic, repo scope only), revoked after each push
- Run node --check before committing any JS changes
- Commit format: one-line summary + bullet body describing what changed

## Related repo
github.com/davidbieri/oil-scenarios — Monte Carlo oil price calculator (React JSX + HTML wrapper landing page)
