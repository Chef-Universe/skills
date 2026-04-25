---
name: simulate-buy
description: Estimate fill, average price, and slippage for a CHEF-denominated buy of any Chef Universe ingredient — no signer, no gas. Useful for sizing trades before execution.
version: 1.0.0
---

# simulate-buy

The agent_bazaar response already includes a pre-computed `liquidity_impact_10k_chef`
field per ingredient (a 10,000 CHEF buy probe). For arbitrary buy sizes, use
the bonding curve math directly with the `curve_steps` array — no extra API
call needed.

## Use the pre-computed 10k probe

```ts
const ing = bazaar.ingredients.find(i => i.ticker === 'cfTRUFFLE')!
const probe = ing.liquidity_impact_10k_chef

console.log(`10k CHEF → ${probe.tokens_received.toFixed(2)} ${ing.ticker}`)
console.log(`avg price: ${probe.avg_price_chef.toFixed(6)} CHEF/token`)
console.log(`slippage:  ${probe.slippage_pct.toFixed(2)}%`)
if (probe.partial_fill) {
  console.log(`⚠️ partial fill — would drain remaining supply`)
}
```

## Custom buy size — walk the steps yourself

`curve_steps` is the full step function (sorted ascending by `range_to`).
Walk from `current_supply` filling each bracket until your CHEF is exhausted.

```ts
function simulateBuy(
  currentSupply: number,
  steps: Array<{ range_to: string; price: string }>,
  chefIn: number,
) {
  // Decimal-string steps → numbers (precision-loss is acceptable for sizing
  // estimates; see Mint Club V2 SDK for precise on-chain math).
  const numericSteps = steps.map(s => ({
    rangeTo: parseFloat(s.range_to),
    price:   parseFloat(s.price),
  }))

  const activeIdx = numericSteps.findIndex(s => s.rangeTo > currentSupply)
  if (activeIdx === -1) return null   // supply exhausted

  const startPrice = numericSteps[activeIdx].price
  let remaining   = chefIn
  let bought      = 0
  let cursor      = currentSupply

  for (let i = activeIdx; i < numericSteps.length && remaining > 0; i++) {
    const step    = numericSteps[i]
    const room    = step.rangeTo - cursor
    if (room <= 0) continue
    const cost    = room * step.price
    if (remaining >= cost) {
      bought   += room
      remaining -= cost
      cursor    = step.rangeTo
    } else {
      const tokensHere = remaining / step.price
      bought   += tokensHere
      cursor   += tokensHere
      remaining = 0
    }
  }

  const spent      = chefIn - remaining
  const avgPrice   = bought > 0 ? spent / bought : 0
  const slippage   = startPrice > 0 ? ((avgPrice - startPrice) / startPrice) * 100 : 0

  return {
    chefIn,
    chefSpent:       spent,
    chefUnspent:     remaining,
    tokensReceived:  bought,
    avgPriceChef:    avgPrice,
    slippagePct:     slippage,
    partialFill:     remaining > 0,
  }
}
```

## When to use this

- **Sizing a trade**: try multiple `chefIn` values and pick one with acceptable slippage
- **Risk caps**: reject the trade if `slippagePct > MY_MAX` before sending
- **UI previews**: show users the projected outcome before they sign

## After this

Once you've decided a size, hand off to `buy-ingredient` for actual execution.
