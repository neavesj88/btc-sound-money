# Bitcoin · Sound Money Calculator

Single-file standalone web app that visualises the value of a Bitcoin stack against USD, AUD, gold (troy oz / gram), and CPI-adjusted fiat over time. Designed to make the inflation/debasement thesis legible at a glance.

Open `index.html` in any browser — desktop, mobile, anywhere. No build step, no server, no API keys.

## Architecture

Everything is in one HTML file:
- Inline CSS (dark vault/terminal aesthetic, Fraunces for headings, JetBrains Mono for numbers)
- Inline JS (vanilla, ~530 lines)
- Two CDN dependencies loaded via `<script src>`:
  - Chart.js 4.4.4
  - chartjs-adapter-date-fns 3

The file is self-contained and works offline after first load (data is cached in `localStorage`).

## Data sources

All three are free, CORS-enabled, and require **no API key**:

| Source | Used for | Endpoint |
|---|---|---|
| `api.kraken.com` | BTC/USD daily OHLC history | `/0/public/OHLC?pair=XBTUSD&interval=1440&since={ts}` |
| `frankfurter.dev` (ECB) | USD → AUD historical FX | `/v1/{start}..{end}?base=USD&symbols=AUD` |
| `freegoldapi.com` | Gold/oz historical (USD, daily) | `/data/latest.json` |

BTC is fetched in USD only; AUD is derived by multiplying BTC/USD × USD-AUD per-day. Gold/gram is derived from gold/oz (÷ 31.1034768). Sats-per-unit views invert: `1e8 / btc_in_quote`.

### Kraken OHLC pagination

Kraken returns up to 720 daily candles per request. For ranges longer than 720 days, the app computes multiple `since` timestamps and fires them as parallel requests, then merges and deduplicates by date. This keeps all ranges fast with no sequential waterfall.

### Caching

The merged BTC series is cached in `localStorage` for 6 hours under keys prefixed `bvc:`. FX and gold responses are cached per-request. To force a refresh: clear site data, or `Shift`+reload. The cache also makes the app work fully offline after the first load.

## Features

- **Stack input**: BTC amount with quick-pick chips (0.001 → 21 ₿)
- **Range**: 90D / 1Y / 3Y / 5Y / MAX (capped at 10 years)
- **View modes**:
  - `Value` — each series on its own y-axis with native units (USD, AUD, oz, g)
  - `Sats / $` — how many sats one unit of the quote currency buys (inflation/debasement thesis view)
  - `% Change` — all series normalised to first day = 0%, single axis for direct growth comparison
- **Scale**: linear / log
- **Toggleable series**: USD, AUD, Gold/oz, Gold/g, USD CPI-adjusted, AUD CPI-adjusted
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

To change home currency, swap `btc_aud` references for another derived series and update FX fetch symbols.

## Files

- `index.html` — the entire app
- `README.md` — this file

Nothing else. No `package.json`, no build, no node_modules. Drop the HTML on any static host (GitHub Pages, Netlify, S3, your own box) or just open the file locally.

## Known caveats

- Kraken history starts around 2014; the MAX range is capped at 10 years. Pre-2014 BTC data is not available.
- Gold history from freegoldapi is annual in the distant past, then daily from around the 1960s onward. Not visible in practice for 5Y/MAX views.
- Frankfurter publishes daily ECB working-day rates; weekends/holidays use nearest-prior trading day (handled by `nearestFXRate`).

## Licence

Personal use, do what you want. Sources retain their own licences.
