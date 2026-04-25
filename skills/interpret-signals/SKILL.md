---
name: interpret-signals
description: Turn Chef Universe agent_bazaar signals (VOLUME_SPIKE_24H, MOMENTUM_12H, LOW_VALUATION, SUPPLY_MILESTONE) into actionable trade hypotheses. Pairs with read-bazaar.
version: 1.0.0
---

# interpret-signals

Each ingredient comes with a `signals[]` array; the global `top_signals[]` is
already ranked across the whole bazaar. This skill explains what each kind
means and how to act on it.

## The four signal kinds

### `VOLUME_SPIKE_24H`
> "24h vol 3.2× daily season avg"

24 h trading volume is at least 2× the per-token daily season average. Typically
fires before or during a sentiment shift — someone (often an agent already!) is
moving size. The score scales 0 (at 2×) → 1 (at 5×+).

**How to act**: high attention. The token is non-quiet today. Read 12h price
change and supply progress to decide direction.

### `MOMENTUM_12H`
> "+24.5% in 12h"

Price has moved up more than +15% over the last 12 h vs $CHEF. Score scales
0 (at +15%) → 1 (at +100%).

**How to act**: trend-following candidate. Confirm with `volume_24h_chef`
not being trivial. Avoid late entries; check `progress` to see how much
supply remains before the next price step.

### `LOW_VALUATION`
> "cap/vol 1.20 (< 1.5)"

Market cap divided by 24 h volume is below 1.5 — i.e., the token traded its
own market cap a lot in 24 h. Often a re-rating opportunity. Score scales
1 (at ratio 0) → 0 (at the threshold 1.5).

**How to act**: deeper-research candidate. Check whether MOMENTUM_12H is
also positive (rotation in progress) or negative (capitulation that may
overshoot).

### `SUPPLY_MILESTONE`
> "supply 92.3% filled"

`progress >= 0.90`. Late stage of the bonding curve — every additional buy
bumps the price hard onto the next step. Score scales 0 (at 0.90) → 1 (at 1.00).

**How to act**: high impact, low capacity. `liquidity_impact_10k_chef.partial_fill`
is likely true here. Size carefully and check `curve_steps` to see how much
room is left at the current step.

## Combining signals on one ticker

If a token has multiple signals, the score in `top_signals` reflects the
single highest-scoring kind for that ticker (multiple kinds can co-fire on
the same token; agent_bazaar surfaces them all in `ingredients[].signals`).

Strong combos:
- `VOLUME_SPIKE_24H` + `MOMENTUM_12H` → real breakout
- `LOW_VALUATION` + `MOMENTUM_12H` → re-rating in progress
- `SUPPLY_MILESTONE` + `VOLUME_SPIKE_24H` → final-stretch climax
- `LOW_VALUATION` alone (no momentum) → patient entry, mean-reversion thesis

## Reliability gating

`VOLUME_SPIKE_24H` and `LOW_VALUATION` only fire when
`volume_source === 'gecko_ohlcv'` for that token. If a ticker only has
`leaderboard_season` data, treat its volume number as a season aggregate, not
a recent window.

`MOMENTUM_12H` requires `price_change_12h_pct` to be non-null (also Gecko-
sourced).

`SUPPLY_MILESTONE` is on-chain only, so it always fires accurately regardless
of Gecko coverage.

## Pseudocode

```ts
const bazaar = await fetch('https://chefuniverse.io/api/v1/agent_bazaar').then(r => r.json())

if (bazaar.top_signals.length === 0) {
  // No actionable signals today — rely on volume leadership instead.
  // bazaar.ingredients[0] is the highest-volume reliable ticker.
  console.log('No signals — defaulting to volume leader:', bazaar.ingredients[0].ticker)
  return
}

const top = bazaar.top_signals[0]
const ing = bazaar.ingredients.find(i => i.ticker === top.ticker)!

// All signals for this ticker
const allSignals = ing.signals.map(s => s.kind).join(', ')
console.log(`${top.ticker}: ${allSignals} (top: ${top.kind} ${top.note})`)
```
