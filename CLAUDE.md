# MSE — Multi-Market Framework for Federal Reserve Policy Analytics

## What this is
Single-file React 18 (CDN + Babel) tool implementing CME FedWatch methodology extended into a full monetary policy analytics suite. Deployed via GitHub Pages at davidbieri.github.io/fed-funds. Current version: v4.5.

## File structure
```
fed_funds_v3.html   <- Main tool (~4,200 lines, single-file React app)
index.html          <- Landing page / documentation wrapper
README.md
CHANGELOG.md
ROADMAP.md
```

## Architecture
- **React 18 + Babel** via CDN -- no build step, no bundler
- **State**: useReducer with 17-action state machine; all derived values in memoized useFinance hook
- **Rendering**: SVG string concatenation injected via dangerouslySetInnerHTML for chart panels; React JSX for interactive controls
- **Live data**: Anthropic API + web search (claude-sonnet-4-6), separate fetch functions per source
- **API key**: injected automatically inside claude.ai; manual sk-ant-... field for standalone use
- **Persistence**: localStorage saves ZQ prices, active scenario, compare scenario

## Editing conventions
- **Prefer targeted str_replace over full rewrites** -- file is ~4,200 lines, full reads cause API 500 errors
- **Scoped context blocks**: name specific functions/components in session preamble, never "read full file"
- **Syntax check after every JS change**: `node --check fed_funds_v3.html`
- **str_replace find strings**: ASCII only -- no \u escapes, no Unicode. Use HTML entities (&#9632; &#8212; etc.) in replacement strings only
- **Input handlers**: always `onchange`, never `oninput` -- prevents re-render loops
- **Arrow functions in SVG helpers**: use `function(){}` not `=>` to avoid minifier edge cases
- **Micro-sessions**: max ~300 lines of changes per session to prevent timeouts

## Tab IDs (fwActive values)
inputs, chain, tree, kalshi, sofr, dashboard, taylor, yieldcurve, riding, financial, balancesheet
Update SET_TAB reducer if adding tabs.

## Color tokens
--c-cme (blue), --c-kal (orange), --c-poly (cyan), --c-dots (purple)
--c-sofr (green), --c-sofr-fwd (light green)
--c-overlay (amber, compare scenario), --c-market (CME blue identity, tab group)
--c-taylor (rose), --c-fin-neutral (teal), --c-stress-hi (red), --c-stress-lo (green)
--c-combined (violet, balance sheet combined stance)
--c-neutral-floor (muted, balance sheet reference lines)

## Reducer actions (17 total)
SET_TAB, SET_LB, SET_SCENARIO, SET_COMPARE, UPDATE_ZQ_PRICE, UPDATE_ZQ_MTG,
UPDATE_ZQ_N, UPDATE_ZQ_M, UPDATE_SR_PRICE, UPDATE_YC_RATE,
APPLY_LIVE_ZQ, APPLY_LIVE_KALSHI, APPLY_LIVE_POLY, APPLY_LIVE_SOFR,
APPLY_LIVE_YC, UPDATE_TAYLOR, UPDATE_TR, SET_RIDE_SELECTED,
APPLY_BALANCE_SHEET, APPLY_FIN_STAB, SET_BSD, SET_FSD, LOAD_SETTINGS

## FOMC schedule (seed)
Apr 28-29, Jun 17-18, Jul 28-29, Sep 16-17, Oct 28-29, Dec 9-10, Jan 27-28, Mar 17-18
Anchor months (no meeting): May, Aug, Nov, Feb

## Git workflow
- Active dev on `dev` branch; `main` touched only at version seal
- Session start: `git checkout dev && git pull origin main`
- Version seal: merge dev->main --no-ff, tag -a vX.Y, push --tags
- Short-lived PAT (classic, repo scope only), revoked after each push
- Run `node --check fed_funds_v3.html` before every commit

## Related repo
github.com/davidbieri/oil-scenarios -- Monte Carlo oil price calculator
