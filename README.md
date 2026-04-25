# Chef-Universe/skills

Agent skills for the **Chef Universe** bazaar — a bonding-curve DEX of 31
ingredient tokens (and counting) denominated in $CHEF, on Base.

## Install

```bash
npx skills add Chef-Universe/skills
```

Compatible with Claude Code, Cursor, Gemini CLI, and any client following the
[agentskills.io](https://agentskills.io/specification) spec.

## Skills

| Skill | What it does |
|---|---|
| `read-bazaar` | Fetch live market state for all 32 assets in one API call |
| `interpret-signals` | Turn `VOLUME_SPIKE` / `MOMENTUM_12H` / `SUPPLY_MILESTONE` / `LOW_VALUATION` into trade hypotheses |
| `simulate-buy` | Estimate fill, average price, and slippage for a CHEF buy — no gas spent |
| `buy-ingredient` | Mint an ingredient token by paying CHEF (via Mint Club V2 SDK) |
| `sell-ingredient` | Burn an ingredient token for CHEF (via Mint Club V2 SDK) |
| `check-portfolio` | Query current ingredient holdings and per-token PnL for an address |
| `read-leaderboard` | Tycoons Arena standings, tiers, and reward thresholds |
| `claim-rewards` | Claim end-of-season Tycoons Arena rewards |

## How writes execute

Trade skills are thin wrappers around the official
**[Mint Club V2 SDK](https://sdk.mint.club)**. Ingredient tokens live on
individual bonding curves (`0xc5a076cad94176c2996B32d8466Be1cE757FAa27` on
Base) and are not routable through generic DEX aggregators — Mint Club's
SDK is the canonical execution layer. Builder code attribution is preserved.

For **CHEF ↔ WETH / USDC** spot swaps (Uniswap-routable), pair this pack with
[`coinbase/agentic-wallet-skills`](https://github.com/coinbase/agentic-wallet-skills) —
its `trade` skill handles standard DEX routing on Base.

## Endpoints

- Public read API: `https://chefuniverse.io/api/v1/agent_bazaar` — 60 s cache, no auth, JSON
- Stack docs + integration guide: `https://chefuniverse.io/for-agents`

## Stack diagram

```
┌─ Read ────────────────────────────────┐
│  GET /api/v1/agent_bazaar             │  → read-bazaar, interpret-signals,
│  (Chef Universe — this pack)          │     simulate-buy, check-portfolio,
│                                       │     read-leaderboard
└───────────────────────────────────────┘
┌─ Write — Ingredient bonds ────────────┐
│  Mint Club V2 SDK                     │  → buy-ingredient, sell-ingredient,
│  (this pack wraps it)                 │     claim-rewards
└───────────────────────────────────────┘
┌─ Write — CHEF spot swaps ─────────────┐
│  coinbase/agentic-wallet-skills       │  → install separately
│  trade / send-usdc / fund / auth      │
└───────────────────────────────────────┘
```

## Built on

- [Mint Club V2](https://mint.club) — bonding-curve protocol
- [Base](https://base.org) — Ethereum L2 by Coinbase
- [GeckoTerminal](https://www.geckoterminal.com) — pool data
- [The Graph / ERC-8004](https://eips.ethereum.org/EIPS/eip-8004) — agent identity

## Status

- Schema version: `1` (stable — additive changes safe; breaking → `/api/v2/…`)
- Repo: `Chef-Universe/skills`
- License: MIT
