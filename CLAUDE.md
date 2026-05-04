# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Single-file financial dashboard (`index.html`) with no build step, no dependencies, and no backend. All HTML, CSS, and JavaScript live in one file. It fetches live market data from public APIs and renders it in the browser.

Hosted via **GitHub Pages** at `https://dmchicago2017.github.io/PaineldeMercado/`. After every push to `main`, the live page updates automatically within 1–2 minutes.

## Deploying changes

```bash
git add index.html
git commit -m "description"
git push
```

No build or compile step. To preview locally, open `index.html` in a browser — but expect some data to fail due to CORS restrictions on `file://`. For accurate testing, push to GitHub Pages and use the live URL.

## Architecture

Everything is in `index.html`. The structure inside it is:

- **CSS** (`<style>`) — all styles, including CSS variables, responsive breakpoints, and animations
- **HTML** (`<body>`) — static shell with placeholder `id` attributes that JavaScript fills in
- **JavaScript** (`<script>`) — data fetching, rendering, and update logic

### Data sources

| Source | What it provides | Fetch method |
|---|---|---|
| Yahoo Finance (`query1.finance.yahoo.com/v8/finance/chart/`) | All tickers, indices, ETFs, FIIs, EUR/BRL, USD/BRL, UST rates | Via CORS proxies (see below) |
| Banco Central do Brasil (`api.bcb.gov.br/dados/serie/`) | Selic (série 432), CDI (série 12), IPCA 12m (série 13522), IGP-M 12m (série 4175) | Direct fetch (no CORS issue) |
| BLS.gov (`api.bls.gov/publicAPI/v2/timeseries/data/CUUR0000SA0`) | US CPI YoY | Via CORS proxies |

### CORS proxy strategy

Yahoo Finance blocks direct browser requests. The code tries three public proxies in sequence, falling back to the next if one fails:

1. `corsproxy.io`
2. `allorigins.win`
3. `codetabs.com`

These are free, shared, and occasionally unreliable. If many tickers show "sem dados", the proxies are likely overloaded — refreshing usually fixes it. When the page is served from `https://` (GitHub Pages), proxies respond much more reliably than from `file://`.

### Key JavaScript functions

- `loadAll()` — entry point, called on page load and on manual refresh; orchestrates all data loading in parallel
- `fetchYahoo(symbol)` — fetches 1Y daily chart from Yahoo Finance via proxies; returns `{ price, dayPct, wkPct, moPct, ytdPct, spark, currency }`
- `loadBCB()` — fetches Selic, CDI, IPCA, IGP-M from the BCB API directly
- `loadMacroYahoo()` — fetches USD/BRL, EUR/BRL, UST 10Y, T-Bill 13W, and CPI
- `sparkline(values)` — generates an inline SVG sparkline for the last 30 days
- `buildSkeletons()` — renders shimmer placeholders in all tables while data loads
- `renderRow(item, data)` / `renderError(item)` — updates a table row with real data or an error state

### Ticker registry

All tracked symbols live in the `TICKERS` object at the top of the `<script>` block, organized by section: `indices`, `br` (Brazilian stocks), `us` (US stocks), `etf`, `fii`. Each entry has `sym` (display ticker), `name`, and `yahoo` (Yahoo Finance symbol).

## Design system

**Fonts:**
- `JetBrains Mono` — body, numbers, all UI text (monospace terminal feel)
- `Fraunces` — section headings and the main `<h1>` title (serif contrast)

**Color palette (CSS variables):**
- Background layers: `--bg` (#0a0e0d) → `--bg-elev` (#111614) → `--bg-card` (#161c1a)
- Borders: `--border` (#1f2725), `--border-strong` (#2a3431)
- Text: `--text` (#e8e6e1), `--text-dim` (#8a8f8c), `--text-faint` (#555a57)
- Accent: `--accent` (#d4a72c) — gold/amber, used for the "Terminal" tag, hover states, loading dot
- Positive: `--green` (#4ade80) with dim background `--green-dim`
- Negative: `--red` (#f87171) with dim background `--red-dim`
- Info: `--blue` (#7dd3fc)

**Layout:** single-column, max-width 1600px, centered. Mobile breakpoint at 768px hides sparkline columns and reduces font sizes.

## Page sections

All sections are in the Macro grid plus five tables:

1. **Macro** (`#macro-grid`) — 10 cells: Selic, CDI, IPCA 12m, USD/BRL, Fed Funds, CPI 12m, UST 10Y, UST 2Y, IGP-M 12m, EUR/BRL
2. **Índices** (`#tbody-indices`) — Ibovespa, IFIX, S&P 500, Nasdaq, Dow Jones
3. **Ações Brasil** (`#tbody-br`) — PETR4, VALE3, EMBJ3, BBAS3, RAPT4
4. **Ações EUA** (`#tbody-us`) — AAPL, MSFT, NVDA
5. **ETFs** (`#tbody-etf`) — BOVA11, VT, BNDW, VNQ, VWRA, AGGU
6. **Fundos Imobiliários** (`#tbody-fii`) — KFOF11

Each table shows: Ticker, Name, Price, Day %, Week %, Month %, YTD %, 30-day sparkline.

## Owner preferences

- Changes should be committed and pushed to `main` so GitHub Pages updates immediately
- The project is intentionally kept as a single `index.html` — do not split into multiple files or introduce a build system
- All monetary values use `pt-BR` locale formatting
- The dark terminal aesthetic is intentional — do not lighten the color scheme
