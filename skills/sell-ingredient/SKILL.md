---
name: sell-ingredient
description: Burn a Chef Universe ingredient token to reclaim $CHEF. Wraps the official Mint Club V2 SDK — bonding-curve burn via the bond contract on Base. Requires a signer.
version: 1.0.0
---

# sell-ingredient

Execute a sell (burn) on a Chef Universe ingredient. Returns $CHEF based on
the bonding curve at burn time. Same bond contract / SDK as `buy-ingredient`.

## Prerequisites

1. The wallet must hold the ingredient token to burn
2. A signer compatible with Mint Club V2 SDK
3. Run `read-bazaar` first to estimate proceeds and check `current_supply`

## Install

```bash
npm install mint.club-v2-sdk
```

## Sell by token amount

```ts
import { mintclub } from 'mint.club-v2-sdk'

await mintclub
  .network('base')
  .bond('0xe52b5AF5156494c28E8E34C54617ebCF0C7EF544')   // cfTRUFFLE token
  .sell({
    amount:    1_500n * 10n ** 18n,    // tokens you're burning (decimals = 18)
    slippage:  5,                       // % tolerance on minimum CHEF received
    recipient: '0xYourAgentWallet',
    onAllowanceSignatureRequest: () => console.log('approving token…'),
    onSignatureRequest:           () => console.log('signing burn…'),
    onSuccess: ({ hash }) => console.log('burn tx:', hash),
  })
```

## Estimate proceeds first

```ts
const estimate = await mintclub
  .network('base')
  .bond(tokenAddress)
  .getSellEstimation({ amount: tokenBalance })

// estimate.refundAmount === CHEF you'd receive
```

## Slippage on the way down

Burning early in the curve (low supply step) returns less CHEF per token than
buying did at a higher step — that's how the curve compensates the protocol.
Plan accordingly:

- If you bought at progress 0.85 and supply has dropped back to 0.50, your
  burn returns step-0.50 prices, not what you paid.
- Use `simulate-buy` style logic on `curve_steps` to estimate the burn refund
  if you need precision.

## Royalties / fees

A burn royalty (lower than mint royalty, typically 0.05% – 0.7%) is taken by
Mint Club V2 protocol. SDK handles it — your `slippage` is enforced post-fee.

## When to use this skill

- Realising profit on a position
- Rebalancing into a different ingredient
- Closing positions before season end (Tycoons Arena rewards lock at season
  boundary; see `claim-rewards`)

## After this

Use `check-portfolio` to confirm balance went to 0 and CHEF balance increased.
If you're rotating into another ingredient, follow up with `buy-ingredient`.
