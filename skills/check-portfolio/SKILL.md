---
name: check-portfolio
description: Query a Chef Universe wallet's current ingredient holdings, $CHEF balance, and per-token PnL. Read-only.
version: 1.0.0
---

# check-portfolio

Read the current holdings of any wallet across the Chef Universe ingredient
set. Useful for risk dashboards, rebalancing logic, and sanity checks after
`buy-ingredient` / `sell-ingredient`.

## Per-token PnL

```
GET https://chefuniverse.io/api/user-pnl?address=0x...&token=0x...
```

Returns per-token cost basis, current value, and PnL percentage based on the
season's mint/burn events.

```ts
const r = await fetch(
  `https://chefuniverse.io/api/user-pnl?address=${wallet}&token=${tokenAddress}`
)
const { balance, costBasisChef, currentValueChef, pnlPct } = await r.json()
```

## Full ingredient sweep

For all 31 ingredient tokens, iterate or read the leaderboard's
`tokenVolumes[]` for the address (already aggregated):

```ts
// Approach 1 — direct ERC-20 balanceOf via viem
import { createPublicClient, http, erc20Abi } from 'viem'
import { base } from 'viem/chains'

const client = createPublicClient({
  chain:     base,
  transport: http('https://base-mainnet.g.alchemy.com/v2/<key>'),
})

const balances = await client.multicall({
  contracts: INGREDIENT_TOKENS.map(t => ({
    address:      t.address,
    abi:          erc20Abi,
    functionName: 'balanceOf',
    args:         [walletAddress],
  })),
  allowFailure: true,
})
```

```ts
// Approach 2 — pull from /api/leaderboard which already has tokenVolumes
const r = await fetch('https://chefuniverse.io/api/leaderboard')
const { board } = await r.json()
const me = board.find(e => e.address.toLowerCase() === wallet.toLowerCase())
console.log('My season volume:', me?.totalVolume, '$CHEF')
console.log('Per token:', me?.tokenVolumes)
console.log('Tier:', me?.tier)   // EMPEROR / TYCOON / ...
```

The leaderboard's `tokenVolumes` is **trading volume**, not current balance —
useful for season-rank contribution but separate from what you hold right now.

## $CHEF balance

```ts
const chefBalance = await client.readContract({
  address:      '0xc4A09803e2e1A491CB3119b891dcf890E3C98B07',  // $CHEF
  abi:          erc20Abi,
  functionName: 'balanceOf',
  args:         [walletAddress],
})
```

## Combining views

Typical agent loop:
1. Fetch CHEF balance + per-ingredient ERC-20 balances
2. For each non-zero balance, fetch `/api/user-pnl` for cost basis
3. Multiply current ingredient balance by `current_price_chef` (from
   `read-bazaar`) for live mark-to-market value

## Privacy note

`/api/user-pnl` and `/api/leaderboard` are public — anyone can query any
address. Don't echo private addresses to public channels you don't control.

## After this

If a position has run its thesis, hand off to `sell-ingredient`. If you're
rebalancing, pair with `buy-ingredient`.
