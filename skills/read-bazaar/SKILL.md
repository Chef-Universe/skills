---
name: read-bazaar
description: Fetch the live Chef Universe market snapshot for all 32 onchain assets in one API call. Returns priced, ranked, and signal-tagged data ready for trading decisions.
version: 1.0.0
---

# read-bazaar

Use this skill whenever you need current market state across the Chef Universe
bazaar — prices, supply, slippage projections, signals, leaderboard standings.
A single GET call returns the full snapshot.

## Endpoint

```
GET https://chefuniverse.io/api/v1/agent_bazaar
```

- **Auth**: none, public
- **Cache**: 60 s server-side, `stale-while-revalidate=120` on CDN
- **Schema**: stable at `version: 1` — additive changes are safe; breaking
  changes go to `/api/v2/...`

## Quick fetch

```ts
const r = await fetch('https://chefuniverse.io/api/v1/agent_bazaar')
const bazaar = await r.json()

console.log(`Cache age: ${bazaar.cache_age_sec}s, block: ${bazaar.block_number}`)
console.log(`CHEF locked across all bonds: ${bazaar.global_asset.total_locked_chef}`)
console.log(`Top signals (${bazaar.top_signals.length}):`)
for (const s of bazaar.top_signals) {
  console.log(`  ${s.kind} on ${s.ticker} — ${s.note} (score ${s.score.toFixed(2)})`)
}
```

## Response shape (key fields)

```ts
{
  generated_at_ms: number   // Unix ms when this payload was built
  block_number:    number   // Base block number the on-chain layer was read at
  cache_age_sec:   number   // 0–60 for hot cache; freshness self-check

  global_asset: {
    symbol:                'CHEF'
    address:               '0x...'
    price_usd:             number | null
    price_weth:            number | null
    marco_polo_vault_chef: number   // CHEF held in our ops vault
    total_locked_chef:     number   // CHEF locked across all ingredient bonds
  }

  ingredients: Array<{
    ticker:                string   // e.g. 'cfTRUFFLE'
    address:               string
    grade:                 1 | 2 | 3 | 4 | 5
    current_price_chef:    number
    current_supply:        number
    max_supply:            number
    progress:              number   // 0–1, supply / max
    market_cap_chef:       number
    curve_steps:           Array<{ range_to: string; price: string }>
    liquidity_impact_10k_chef: {
      tokens_received: number
      avg_price_chef:  number
      slippage_pct:    number
      partial_fill:    boolean
    }
    volume_24h_chef:       number
    volume_source:         'gecko_ohlcv' | 'leaderboard_season' | 'none'
    price_change_24h_pct:  number | null
    price_change_12h_pct:  number | null
    signals: Array<{ kind, score, note }>
  }>

  top_signals: Array<{ ticker, kind, score, note }>   // ranked, capped at 10
}
```

## When to call this skill

- Always call **before** any trade decision so you have current state
- Cache locally for ≤ 60 s — re-fetching faster gives no fresher data
- For a single-asset look-up, find the entry: `bazaar.ingredients.find(i => i.ticker === 'cfTRUFFLE')`

## Reliability notes

- `volume_source: 'gecko_ohlcv'` → the 24 h figure is a real rolling window
- `volume_source: 'leaderboard_season'` → token isn't well-indexed by Gecko;
  the 24 h figure is actually a season aggregate. Don't compare it to other
  tokens' real 24 h numbers without normalizing.
- `cache_age_sec`: clamp tightly (`< 70`) if your strategy is sensitive to
  staleness.

## After this

Pipe the result into `interpret-signals` to turn raw data into hypotheses,
or pull a single ticker for `simulate-buy` / `buy-ingredient`.
