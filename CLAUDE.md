# MSE: A Multi-Market Framework for Federal Reserve Policy Analytics

## What this is
Single-file HTML tool (fed_funds_v3.html) implementing CME FedWatch methodology (30-day ZQ
futures chain → unconditional probability lattice) with Kalshi + Polymarket prediction market
comparison, SOFR strip, Dashboard (incl. Policy Confidence Index), Nelson-Siegel yield curve,
Taylor Rule suite (five variants incl. Judd-Rudebusch ECM), Bieri-Chincarini (2005)
riding-the-yield-curve calculator, and Financial Stability tab (BIS/Mehrling framework).
Deployed via GitHub Pages at davidbieri.github.io/fed-funds.

## File structure
```
fed_funds_v3.html                <- Main tool (~3,500 lines, React 18 CDN)
fed_funds_probability_tree.html  <- Legacy v1 vanilla JS (archive, do not edit)
index.html                       <- Landing page (launches fed_funds_v3.html in iframe modal)
README.md
CHANGELOG.md
CLAUDE.md
claude_code_scripts.md           <- Implementation archive: Phases 1-4 complete
```

## Architecture
- **React 18 CDN, zero dependencies** -- no build step, no bundler, open directly in browser
- **State**: `useReducer` with 21-action state machine; all derived values in `useFinance` memoized hook
- **Rendering**: SVG string concatenation + React JSX components; 10 tabs render from shared state
- **Live data**: Anthropic API + web search (claude-sonnet-4-20250514), separate refresh buttons per source
- **API key**: injected automatically inside claude.ai; manual sk-ant-... field for standalone use
- **Signal chain**: ZQ prices → EFFR chain → Prob. tree → Pred. markets (PCI) → SOFR strip → Dashboard → Yield Curve → Taylor Rule (JR ECM signal) → Riding Calculator → Financial Stability

## Editing conventions
- **Prefer targeted str_replace over full rewrites** -- file is large (~3,500 lines), full rewrites risk regression
- **Syntax check after every JS change**: `node --check fed_funds_v3.html`
- **Non-ASCII box-drawing chars in JS comments cause node --check false positives** -- use plain ASCII `---` not box chars
- **Input handlers**: always `onchange`, never `oninput` -- prevents re-render loops
- **Tab IDs**: inputs, chain, tree, kalshi, sofr, dashboard, taylor, yieldcurve, riding, financial -- update SET_TAB reducer if adding tabs

## Color tokens
--c-cme blue, --c-kal orange, --c-poly cyan, --c-dots purple, --c-sofr green, --c-sofr-fwd light green
--c-taylor rose (#e11d48), --c-overlay amber (#f59e0b), --c-riding violet (implicit #8b5cf6)
--c-fin-neutral teal (#0d9488), --c-stress-hi red (#dc2626), --c-stress-lo green (#16a34a)

## FOMC schedule (seed, corrected)
Apr 28-29 (Powell's last), Jun 17-18, Jul 28-29, Sep 16-17, Oct 28-29, Dec 9-10, Jan 27-28, Mar 17-18
Anchor months (no meeting): May, Aug, Nov, Feb

## Git workflow
- Active development on `dev` branch; `main` is only updated at version seal
- Short-lived personal access tokens (classic, repo scope only), revoked after each push
- Run node --check before committing any JS changes
- Commit format: one-line summary + bullet body describing what changed

## Documentation discipline

### Per-session (before every commit)
Add an entry under `## [Unreleased]` in CHANGELOG.md describing what the session changed.
Minimum: tab reference, feature name, key state fields or math functions added.
Do NOT update README.md or index.html mid-session -- those are version-boundary tasks.

### Version seal (once per phase, in the final session)
1. Promote `[Unreleased]` → `[X.0]` with date in CHANGELOG
2. Update README: tab count, scenario tables (from SCENARIOS const), methodology, version badge, citations
3. Update index.html: sync tab cards, scenario section, data sources, version strings
4. Update CLAUDE.md: tab IDs, color tokens, action count if changed
5. Verify `<title>` and h2 in fed_funds_v3.html match README tool name
6. `git checkout main && git merge dev && git tag -a vX.0 -m "Phase X complete"`

### Canonical sources (always read from file, never from memory)
- Version number: CHANGELOG.md `[X.0]` heading
- Scenario list: `SCENARIOS` const in fed_funds_v3.html
- Tab list: `TABS` array in App() component of fed_funds_v3.html
- Color tokens: `:root` CSS block in fed_funds_v3.html

## Related repo
github.com/davidbieri/oil-scenarios -- Monte Carlo oil price calculator (React JSX + HTML wrapper)
