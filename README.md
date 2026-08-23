# Krónan price data

Daily prices for a fixed 100-item Icelandic grocery basket, collected from the
[Krónan public API](https://api.kronan.is) and published here as a static feed.

Generated automatically once a day. Every file is regenerated on each run, so
URLs are stable — fetch, don't scrape.

## Files

| File | Contents |
|---|---|
| `meta.json` | Coverage, schema version, generation timestamp |
| `basket.json` | The 100 tracked items and their pinned SKUs |
| `latest.json` | Most recent observation for every item |
| `series/<item_id>.json` | Full price history for one item |
| `prices.csv` | Every observation ever recorded, one row per (date, SKU) |

Base URL:

```
https://raw.githubusercontent.com/FffD/krobolgan-data/main/
```

`Access-Control-Allow-Origin: *` is set, so browser apps can fetch these
directly without a proxy.

```js
const latest = await fetch(
  "https://raw.githubusercontent.com/FffD/krobolgan-data/main/latest.json"
).then((r) => r.json());
```

## Reading the prices

All amounts are integers in ISK, except `unit_price` which is a float.

- **`shelf_price`** — the price before any discount.
- **`effective_price`** — what you actually pay that day. Use this for a
  cost-of-living measure; use `shelf_price` to exclude promotions.
- **`unit_price`** — price per `unit` (`KG`, `LTR`, `STK`, …). **Prefer this
  for measuring inflation.** It is the only field that stays comparable when a
  package size changes: a 500 g pack shrinking to 450 g at an unchanged price
  leaves `effective_price` flat while `unit_price` correctly steps up.
- **`status`** — `ok`, or `missing` when the SKU was delisted and returned no
  price that day. Missing days are recorded explicitly rather than omitted, so
  a gap in the data is distinguishable from a day the collector never ran.

Ten of the items are sold by weight (`charged_by_weight: true` — meat, counter
cheese). For those, `shelf_price` is Krónan's estimate for a *nominal* pack
weight and moves whenever that assumed weight is revised, which is not a price
change. `unit_price` is the real series for those items.

## No index is included

Deliberately. Turning these prices into an index requires choices about
weighting, chaining, and how to splice a series across a product substitution —
and baking one set of choices into the feed would quietly make it the only one
anyone uses. The raw matched-product series is here; the methodology is yours.

## Caveats

- Prices are Krónan's **home-delivery** prices, which may differ from in-store
  shelf prices. Consistent over time, so trends are valid.
- One store chain only. This is not a national CPI.
- Products get delisted. When that happens the SKU is retired and a
  replacement pinned, which introduces a discontinuity in that item's series —
  check `sku` changes before treating a series as continuous.

## Source

Collector lives in a separate private repository. Data here is published under
[CC0](https://creativecommons.org/publicdomain/zero/1.0/); the underlying price
information belongs to Krónan.
