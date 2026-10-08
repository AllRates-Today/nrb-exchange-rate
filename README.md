# Nepal Rastra Bank Exchange Rates API — nrb-exchange-rate

[![npm version](https://img.shields.io/npm/v/nrb-exchange-rate.svg)](https://www.npmjs.com/package/nrb-exchange-rate)
[![license](https://img.shields.io/npm/l/nrb-exchange-rate.svg)](https://github.com/AllRates-Today/nrb-exchange-rate/blob/main/LICENSE)
[![zero dependencies](https://img.shields.io/badge/dependencies-0-brightgreen.svg)](https://www.npmjs.com/package/nrb-exchange-rate)
[![TypeScript](https://img.shields.io/badge/TypeScript-types%20included-3178C6.svg)](https://www.typescriptlang.org/)

**Official Nepal Rastra Bank (Nepal) daily exchange rates for Node.js and TypeScript. The published central bank rates behind tax filings, customs valuations, audits, and compliant invoicing — not market estimates, but the numbers Nepal Rastra Bank itself prints, every business day.**

## 🚀 Why this client?

- 🏛️ **Official published rates** — Nepal Rastra Bank's own table, with the publisher's own `rate_date` on every response
- 📅 **History back to 1998** — point-in-time tables and daily series for any past date
- 🔀 **Published vs derived, always flagged** — computed inverse/cross pairs carry `derived: true`, never mixed with official prints
- ⚡ **Zero dependencies** — pure ESM + CJS over global `fetch`; Node 18+, Bun, Deno, and edge runtimes
- 🔷 **Type-safe** — full TypeScript definitions shipped with the package
- 🧾 **Compliance-grade metadata** — `rate_type`, publication date, and source disclaimer on every response

> **Official rate, not mid-market:** every value here is a number Nepal Rastra Bank itself published, fixed once printed and carrying the central bank's own `rate_date` — what filings and audits require. Need the live mid-market rate for pricing or display instead? Use the [mid-market API](https://allratestoday.com/docs/) or [`@allratestoday/sdk`](https://www.npmjs.com/package/@allratestoday/sdk). The two can diverge by several percent.

## ⚡ Try it without a key

The latest Nepal Rastra Bank table is also served keyless, CORS-open and edge-cached, for evaluation, embeds and AI agents:

```bash
curl "https://allratestoday.com/api/open/central-bank/nrb?source=USD&target=NPR"
```

```js
const r = await fetch('https://allratestoday.com/api/open/central-bank/nrb').then((x) => x.json());
console.log(r.rate_date, r.rates.length); // the central bank's latest published table, no key
```

The open endpoint serves the *latest* table only and asks for a visible attribution link. The client below uses the keyed API, which adds point-in-time tables, history, and CSV/XML/Excel output.

## 🔑 Get your API key

Get a free API key at [allratestoday.com/register](https://allratestoday.com/register) — no credit card required. Latest rates are on every plan, including free.

## 📦 Installation

```bash
npm install nrb-exchange-rate
```

```bash
yarn add nrb-exchange-rate
```

```bash
pnpm add nrb-exchange-rate
```

Requires Node 18+ (global `fetch`); also runs on Bun, Deno and edge runtimes. Also published under the org scope as [`@allratestoday/nrb-exchange-rate`](https://www.npmjs.com/package/@allratestoday/nrb-exchange-rate) — same code, same versions.

## 🏁 Quick start

```js
import { getRate } from 'nrb-exchange-rate';

const pair = await getRate('USD', 'NPR', { apiKey: 'art_live_...' });
console.log(pair.rate, pair.rate_date); // the official Nepal Rastra Bank rate, on the central bank's own date
```

## 📚 API reference

- [Latest pair rate](#latest-pair-rate) — one pair from the latest published table
- [Full published table](#full-published-table) — everything the central bank printed, in one call
- [Table for a date](#table-for-a-date) — the official table for an invoice or filing date
- [Daily time series](#daily-time-series) — one pair across a date range

---

### Latest pair rate

Free plan and up. Pairs the central bank does not print directly are resolved from its table and flagged (see *Published vs derived rates* below).

```js
const pair = await getRate('USD', 'NPR', { apiKey: 'art_live_...' });
```

**Response:**

```javascript
{
  bank: 'nrb',
  name: 'Nepal Rastra Bank',
  rate_date: '2026-10-08',   // Nepal Rastra Bank's own publication date
  source: 'USD',
  target: 'NPR',
  rate: 155.15,
  rate_type: 'sell',
  derived: false,
  method: 'published',
  disclaimer: 'Official rates as published by the named central bank. On weekends/holidays the most recent published rate_date is returned.'
}
```

### Full published table

Free plan and up. The complete table for the latest publication date.

```js
import { getLatestRates } from 'nrb-exchange-rate';

const table = await getLatestRates({ apiKey: 'art_live_...' });
console.log(table.rate_date, table.rates.length);
```

**Response:**

```javascript
{
  bank: 'nrb',
  name: 'Nepal Rastra Bank',
  rate_date: '2026-10-08',
  rates: [
    { "base": "USD", "quote": "NPR", "type": "sell", "value": 155.15 },
    { "base": "USD", "quote": "NPR", "type": "buy", "value": 154.55 },
    // … the rest of the published table (22 currencies vs NPR)
  ],
  disclaimer: '…'
}
```

### Table for a date

Paid plans. The official table for any date since 1998 — weekends and holidays return the most recent published date, flagged via `published_on_requested_date`, which is exactly the in-force rate a filing needs.

```js
import { getRatesForDate } from 'nrb-exchange-rate';

const day = await getRatesForDate('2026-06-30', { apiKey: 'art_live_...' });
// Optionally narrow to one pair:
const one = await getRatesForDate('2026-06-30', { apiKey: 'art_live_...', source: 'USD', target: 'NPR' });
```

**Response:**

```javascript
{
  bank: 'nrb',
  requested_date: '2026-06-30',
  rate_date: '2026-06-30',                // the date actually published
  published_on_requested_date: true,      // false when a weekend/holiday fell back
  rates: [ /* the full table for that date */ ],
  disclaimer: '…'
}
```

### Daily time series

Paid plans. One resolved rate per publication date — ready for charting, revaluation runs, or audit workpapers.

```js
import { getHistory } from 'nrb-exchange-rate';

const series = await getHistory(
  { source: 'USD', target: 'NPR', from: '2026-01-01', to: '2026-10-08' },
  { apiKey: 'art_live_...' }
);
```

**Response:**

```javascript
{
  bank: 'nrb',
  source: 'USD',
  target: 'NPR',
  from: '2026-01-01',
  to: '2026-10-08',
  count: 152,
  rates: [
    // one entry per publication date
    { date: '2026-10-08', rate: 155.15, rate_type: 'sell', derived: false, method: 'published' },
    // …
  ],
  disclaimer: '…'
}
```

Pass `{ symbol: 'USD' }` instead of `source`/`target` to get the raw published rows for one currency (all rate types, no pair resolution).

---

## 🗺️ Currencies covered

Nepal Rastra Bank currently publishes rates covering **22 currencies** against the NPR (as of the latest table):

🇦🇪 `AED` · 🇦🇺 `AUD` · 🇧🇭 `BHD` · 🇨🇦 `CAD` · 🇨🇭 `CHF` · 🇨🇳 `CNY` · 🇩🇰 `DKK` · 🇪🇺 `EUR` · 🇬🇧 `GBP` · 🇭🇰 `HKD` · 🇮🇳 `INR` · 🇯🇵 `JPY` · 🇰🇷 `KRW` · 🇰🇼 `KWD` · 🇲🇾 `MYR` · 🇴🇲 `OMR` · 🇶🇦 `QAR` · 🇸🇦 `SAR` · 🇸🇪 `SEK` · 🇸🇬 `SGD` · 🇹🇭 `THB` · 🇺🇸 `USD`

## 🏛️ Source

Nepal Rastra Bank is Nepal's central bank and publishes the official buying and selling rates for around 22 currencies against the Nepalese rupee each business day. Because the rupee is pegged to the Indian rupee at a fixed 1.60, the INR rate is a constant and the rest are set from it, and these are the rates Nepali customs, tax filings and audited accounts are expected to cite.

- Publisher's own page: [Foreign exchange rates](https://www.nrb.org.np/forex/) · [www.nrb.org.np](https://www.nrb.org.np)
- Publication: every business day; the exact schedule, freshness status and any current delay are on the [Nepal Rastra Bank rates page](https://allratestoday.com/central-bank-rates-api/nrb/)
- Values are stored unmodified, with the publisher's own `rate_date` on every row — see the [methodology](https://allratestoday.com/official-rates-methodology/)

## 🧭 Reading the numbers

- `value` is always **quote currency per 1 unit of base currency** (`base: "EUR", quote: "USD", value: 1.15` means 1 EUR = 1.15 USD).
- Nepal Rastra Bank quotes **NPR per 1 unit of foreign currency** (e.g. `base: "USD", quote: "NPR"` means NPR per one US dollar).
- Need the other way round? Ask `getRate(target, source)` and the API inverts or crosses for you, flagged `derived: true` — never divide a published rate yourself in a compliance workflow.
- `rate_type` tells you which of the central bank's series a row belongs to (`sell` here); some publishers print buy/sell or several fixings for the same pair.

## 🧩 ERP & accounting systems

Loading the official Nepal Rastra Bank rate into an accounting system is a supported workflow, not a hack. Step-by-step guides with the direction each system expects:

- [Dynamics 365 Business Central](https://allratestoday.com/docs/integrations/business-central/) — built-in Currency Exchange Rate Service, no code
- [Xero](https://allratestoday.com/docs/integrations/xero/) · [QuickBooks Online](https://allratestoday.com/docs/integrations/quickbooks/) · [SAP S/4HANA and ECC](https://allratestoday.com/docs/integrations/sap/) · [Odoo](https://allratestoday.com/docs/integrations/odoo/)

The same keyed endpoints return `?format=csv`, `?format=xml` and `?format=xlsx`, and accept the key as `?api_key=` on the URL for importers that cannot send headers:

```bash
curl "https://allratestoday.com/api/v1/central-bank/nrb/latest?format=xml&api_key=art_live_..."
```

## 🤖 AI agents

- MCP server: `npx -y @allratestoday/central-bank-mcp` (stdio) or the hosted endpoint `https://allratestoday.com/api/mcp` — tools for official rates, history, cross-bank comparison and publication calendars
- Already using the general SDK or MCP server? Since 2026-10-01 [`@allratestoday/sdk`](https://www.npmjs.com/package/@allratestoday/sdk) 1.4+ has `officialRates('nrb')` and [`@allratestoday/mcp-server`](https://www.npmjs.com/package/@allratestoday/mcp-server) 0.6+ has a `get_official_rates` tool — both return this source's latest table with no key, so you can add it without a second dependency
- Claude Code plugin (no key): `/plugin marketplace add AllRates-Today/claude-code-plugin` then `/plugin install allratestoday@allratestoday` — bundles both MCP servers plus an `/official-rate nrb ...` command
- Machine-readable site guide: [llms.txt](https://allratestoday.com/llms.txt) · [for-ai-agents](https://allratestoday.com/for-ai-agents/)

## ⚖️ Published vs derived rates

If Nepal Rastra Bank does not print a pair directly, the API resolves it from the central bank's own table and says so — official and computed values are never confused:

| `method` | `derived` | Meaning |
| --- | --- | --- |
| `published` | `false` | The central bank printed this pair directly |
| `inverse` | `true` | Computed as 1 ÷ the published opposite direction |
| `cross` | `true` | Computed via NPR from two published rates |

## 🛡️ Error handling

Errors are thrown as `Error` with `status` (HTTP code) and `body` (the API's JSON error) attached:

```js
try {
  const pair = await getRate('USD', 'XXX', { apiKey: 'art_live_...' });
} catch (err) {
  console.log(err.message); // human-readable reason
  console.log(err.status);  // e.g. 404
}
```

| Status | Meaning |
| ------ | ------- |
| — | Missing `apiKey` (thrown before any request) |
| `400` | Malformed date or parameters |
| `401` | Invalid API key |
| `403` | Endpoint needs a [paid plan](https://allratestoday.com/pricing/) (historical dates & series) |
| `404` | Pair or date range not covered by Nepal Rastra Bank |
| `429` | Monthly quota exceeded |

## 🔷 TypeScript

Full definitions ship with the package — no `@types` install:

```ts
import type { LatestRates, PairRate, DatedRates, RateEntry, HistoryQuery, RequestOptions } from 'nrb-exchange-rate';
```

## 📦 CommonJS

```javascript
const { getRate } = require('nrb-exchange-rate');

getRate('USD', 'NPR', { apiKey: 'art_live_...' }).then((pair) => console.log(pair.rate));
```

## 💡 Quota tips

- Rates change once per business day — cache the published table locally and a small monthly quota goes a long way.
- Every request counts toward your AllRatesToday quota, shared across all AllRatesToday endpoints on your key.

## 📖 Methods reference

| Method | Plan | Description |
| ------ | ---- | ----------- |
| `getRate(source, target, { apiKey })` | Free | Latest rate for one pair, resolved from the published table |
| `getLatestRates({ apiKey })` | Free | The central bank's full latest published table |
| `getRatesForDate(date, { apiKey, source?, target? })` | Paid | The official table (or one pair) for a YYYY-MM-DD date |
| `getHistory({ symbol \| source+target, from?, to? }, { apiKey })` | Paid | Daily series since 1998 |

## 📥 Bulk data (no key)

Need the whole archive rather than an API call? The same published tables are mirrored daily as open data:

- Hugging Face: [AllRates/central-bank-exchange-rates](https://huggingface.co/datasets/AllRates/central-bank-exchange-rates) — one CSV per institution (`rates/nrb.csv`)
- Kaggle: [allratestoday/central-bank-exchange-rates](https://www.kaggle.com/datasets/allratestoday/central-bank-exchange-rates)
- CDN JSON: `https://cdn.jsdelivr.net/gh/AllRates-Today/central-bank-exchange-rates@main/data/nrb/latest.json`

## 🔗 Links

- [Nepal Rastra Bank rates page](https://allratestoday.com/central-bank-rates-api/nrb/) — live table, publication cadence, FAQ
- [All central bank sources](https://allratestoday.com/central-bank-rates-api/)
- [Package docs on the site](https://allratestoday.com/docs/sdk/nrb-exchange-rate/) · [ERP integration guides](https://allratestoday.com/docs/integrations/)
- [API documentation](https://allratestoday.com/docs/#central-bank) · [Interactive reference](https://allratestoday.com/api-reference/) · [Methodology](https://allratestoday.com/official-rates-methodology/)
- [Register (free)](https://allratestoday.com/register) · [Pricing](https://allratestoday.com/pricing/)
- [GitHub](https://github.com/AllRates-Today/nrb-exchange-rate)

## 📜 License

MIT
