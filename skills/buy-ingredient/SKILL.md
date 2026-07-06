---
name: buy-ingredient
description: Mint a Chef Universe ingredient token by paying $CHEF. Wraps the official Mint Club V2 SDK — bonding-curve mint via the bond contract on Base. Requires a signer.
version: 1.0.0
---

# buy-ingredient

Execute an actual buy on a Chef Universe ingredient. Reserve token is
$CHEF; bond contract is `0xc5a076cad94176c2996B32d8466Be1cE757FAa27`
on Base. This skill wraps the official Mint Club V2 SDK — there is no
DEX-aggregator path for ingredient tokens.

## Prerequisites

1. A funded wallet on Base with $CHEF balance for the buy + ETH for gas
2. A signer (viem WalletClient, ethers Signer, or an RPC provider that the
   Mint Club SDK accepts — see [sdk.mint.club](https://sdk.mint.club))
3. Run `read-bazaar` first to know:
   - Token address
   - Current liquidity impact for your intended size

## Install

```bash
npm install mint.club-v2-sdk
```

## Buy by token amount

```ts
import { mintclub } from 'mint.club-v2-sdk'

await mintclub
  .network('base')
  .bond('0xe52b5AF5156494c28E8E34C54617ebCF0C7EF544')   // cfTRUFFLE token
  .buy({
    amount:    1_500n * 10n ** 18n,    // tokens you want (decimals = 18)
    slippage:  5,                       // % tolerance
    recipient: '0xYourAgentWallet',
    onAllowanceSignatureRequest: () => console.log('approving CHEF…'),
    onSignatureRequest:           () => console.log('signing mint…'),
    onSuccess: ({ hash }) => console.log('mint tx:', hash),
  })
```

## Buy by CHEF spend (use simulate-buy first)

```ts
import { mintclub } from 'mint.club-v2-sdk'

// 1. Estimate first
const estimate = await mintclub
  .network('base')
  .bond(tokenAddress)
  .getBuyEstimation({ reserveAmount: 10_000n * 10n ** 18n })

// estimate.amount === tokens you'd receive

// 2. Then mint that exact token amount with a slippage cap
await mintclub
  .network('base')
  .bond(tokenAddress)
  .buy({
    amount:    estimate.amount,
    slippage:  5,
    recipient: signerAddress,
  })
```

## Sizing checklist before calling

Pull these from `read-bazaar` for your target ticker:
- `liquidity_impact_10k_chef.partial_fill` — if true at 10k CHEF, your size
  is risky; reduce or split.
- `progress` — if `>= 0.90` (`SUPPLY_MILESTONE`), each step bumps price hard;
  `slippage` of 1–3% may be too tight.
- `volume_source` — if `'indexer_24h'` (Gecko had no candles), you still get a
  real 24 h volume figure but `price_change_*_pct` is `null`, so you have no
  real-time price-trend signal; consider a smaller size.

## Royalties / fees

Mint Club V2 charges a star-based mint royalty (0.2% – 1.4%, varies by token
tier). The Mint Club SDK handles this transparently — your `slippage`
parameter is enforced by the contract on top of fees.

## What this skill does NOT do

- **Does not swap CHEF → WETH/USDC.** For that, use
  [`coinbase/agentic-wallet-skills`](https://github.com/coinbase/agentic-wallet-skills)
  `trade` skill (Uniswap routing).
- **Does not fund the wallet.** Use Coinbase skills `fund` for USDC onramp,
  then swap to CHEF via `trade`.

## Attribution

Builder code (ERC-8021 `bc_qh2d9gah`) is appended to calldata so the trade
is credited to Chef Universe in indexers and analytics. The SDK injects this
automatically; do not strip it.

## After this

Run `check-portfolio` for the recipient address to confirm new holdings, or
go straight to `read-leaderboard` to see if your trade moved your tier.
