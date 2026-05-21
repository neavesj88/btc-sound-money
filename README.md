# Bitcoin · Sound Money Calculator

Single-file standalone web app that visualises the value of a Bitcoin stack against USD, AUD, gold (troy oz / gram), CPI-adjusted fiat, and US M2 money supply over time. Designed to make the inflation/debasement thesis legible at a glance.

Open `index.html` in any browser — desktop, mobile, anywhere. No build step, no server, no API keys.

## Architecture

Everything is in one HTML file:
- Inline CSS (dark vault/terminal aesthetic, Fraunces for headings, JetBrains Mono for numbers)
- Inline JS (vanilla, ~520 lines)
- Two CDN dependencies loaded via `<script src>`:
  - Chart.js 4.4.4
  - chartjs-adapter-date-fns 3

The file is self-contained and works offline after first load (data is cached in `localStorage`).

## Data sources

All three are free, CORS-enabled, and require **no API key**:

| Source | Used for | Endpoint |
|---|---|---|
| `charts.bitcoin.com` | BTC/USD daily history + US M2 money supply (one call) | `/api/v1/charts/m2?interval=daily&timespan={30d\|1y\|2y\|5y\|all}` |
| `frankfurter.dev` (ECB) | USD → AUD historical FX | `/v1/{start}..{end}?base=USD&symbols=AUD` |
| `freegoldapi.com` | Gold/oz historical (USD, daily after ~1968) | `/data/latest.json` |

BTC is fetched in USD only; AUD is derived by multiplying BTC/USD × USD-AUD per-day. Gold/gram is derived from gold/oz (÷ 31.1034768). Sats-per-unit views invert: `1e8 / btc_in_quote`.

### Why `charts.bitcoin.com` instead of CoinGecko

CoinGecko's free public API now restricts historical data to the last 365 days, and the `interval=daily` parameter is paid-only. `charts.bitcoin.com` has no key, no rate limit, and supports `timespan=5y` / `all`. The `m2` endpoint conveniently returns both BTC price and US M2 money supply in `data.combined` as `[{timestamp, btcPrice, m2Billions, ...}, ...]`.

### Caching

All API responses are cached in `localStorage` for 6 hours under keys prefixed `bvc:`. To force a refresh: clear site data, or `Shift`+reload. The cache also makes the app work fully offline after the first load.

## Features

- **Stack input**: BTC amount with quick-pick chips (0.001 → 21 ₿)
- **Range**: 90D / 1Y / 3Y / 5Y / MAX
- **View modes**:
  - `Value` — each series on its own y-axis with native units (USD, AUD, oz, g, $T for M2)
  - `Sats / $` — how many sats one unit of the quote currency buys (inflation/debasement thesis view)
  - `% Change` — all series normalised to first day = 0%, single axis for direct growth comparison
- **Scale**: linear / log
- **Toggleable series**: USD, AUD, Gold/oz, Gold/g, USD CPI-adjusted, AUD CPI-adjusted, US M2 supply
- **Live "now" strip**: current stack value in all four primary units
- **Time machine**: "If I bought X AUD on date Y" → BTC acquired, current value, multiplier, CAGR, and inflation-adjusted real multiplier
- **Purchasing power table**: stack value translated to AUD reference prices (bread, fuel, schooner, median Perth rent/house, etc.)

### CPI adjustment

CPI lines use long-run smoothed annual rates (USD ≈ 3.2%/y, AUD ≈ 3.0%/y) compounded against time-distance from today. Not month-exact CPI prints — directionally honest, not audit-grade. Constants live in the `INFLATION` object.

## Customising

Most things you'd want to tweak are top-of-script constants:

```js
const G_PER_OZ = 31.1034768;
const CACHE_TTL_MS = 6 * 60 * 60 * 1000;
const INFLATION = { USD: 0.032, AUD: 0.030 };
const SERIES_META = { ... };               // labels, colours, dash styles per line
const PURCHASING_POWER_ITEMS_AUD = [ ... ]; // edit reference prices here
```

Other localised assumptions:
- Locale is `en-AU` for number/date formatting
- Purchasing-power references are Perth-skewed (median rent, median Perth house, Perth unleaded)
- AUD is the "home" currency for the time-machine input

To change home currency, swap `btc_aud` references for another derived series and update FX fetch symbols. To extend to other crypto, switch the `charts.bitcoin.com` source for an equivalent — note the response shape is BTC-specific in the m2 endpoint.

## Files

- `index.html` — the entire app
- `README.md` — this file

Nothing else. No `package.json`, no build, no node_modules. Drop the HTML on any static host (GitHub Pages, Netlify, S3, your own box) or just open the file locally.

## Known caveats

- `m2Billions` from charts.bitcoin.com is monthly, so the M2 line shows step-changes between FRED publication dates rather than smooth growth.
- Gold history is annual until ~mid-1900s, then more frequent. The 5y/MAX views use the recent dense daily data, so this isn't visible in practice.
- Frankfurter publishes daily ECB working-day rates; weekends/holidays use nearest-prior trading day (handled by `nearestFXRate`).

## Licence

Personal use, do what you want. Sources retain their own licences.
