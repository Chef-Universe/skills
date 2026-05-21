---
name: claim-rewards
description: Claim end-of-season Tycoons Arena $CHEF rewards. Window opens on the 1st of each month UTC and stays open for a limited duration.
version: 1.0.0
---

# claim-rewards

At the end of each Tycoons Arena season (last UTC second of each month),
the leaderboard freezes and reward pools allocate to the top tiers
(`EMPEROR`, `TYCOON`, `GRAND_MERCHANT`, `MERCHANT`, `APPRENTICE`).

## Check eligibility

```ts
// Pull last month's frozen final standings
const r = await fetch('https://chefuniverse.io/api/leaderboard?season=prev')
const data = await r.json()
if (data.error) {
  console.log('Previous season snapshot not yet available.')
  return
}

const me = data.board.find(e => e.address.toLowerCase() === myWallet.toLowerCase())
if (!me) {
  console.log('Not in the top 100 last season. No reward.')
  return
}

console.log(`Final rank: ${me.rank}, tier: ${me.tier}, season volume: ${me.totalVolume}`)
```

## Claim window

The claim window is a fixed period after season end (typically one week,
configurable in `lib/tycoons.constants.ts:getClaimWindowEnd`). After the
window expires, unclaimed rewards return to the protocol pool.

To check the deadline:

```ts
// The leaderboard's epoch + the constants module on chefuniverse-web define
// the exact claim end. Conservative rule: claim within 7 days of season end.
```

## Execute claim

The claim is an onchain transaction. The exact entry point depends on the
Tycoons Arena settlement implementation in production.

> **Status**: This skill is a placeholder for the production claim flow.
> While the season-rewards smart contract is being finalised, the canonical
> claim path is the in-app UI at
> [chefuniverse.io/tycoons](https://chefuniverse.io/tycoons) — the
> "Reveal & Claim" modal appears for eligible addresses.

When the headless API surfaces, this skill will be updated with the contract
ABI + signer flow. Subscribe to repo changes:
[github.com/Chef-Universe/skills](https://github.com/Chef-Universe/skills).

## After this

Once claimed, your $CHEF balance increases. Keep it for next season's volume
or convert via `coinbase/agentic-wallet-skills` `trade` skill (CHEF/WETH on
Uniswap).
