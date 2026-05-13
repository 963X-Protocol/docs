# X963 Tokenomics

## Token facts

| Field | Value |
|-------|-------|
| **Name** | 963X |
| **Symbol** | X963 |
| **Standard** | ERC-20 (ERC20Burnable) |
| **Network** | Arbitrum One (chainId 42161) |
| **Contract** | [`0xe257878652bA7f4E9C57B688370a8d7875484ed7`](https://arbiscan.io/address/0xe257878652bA7f4E9C57B688370a8d7875484ed7) |
| **Decimals** | 18 |
| **Max supply** | 1,000,000,000 (1B) |
| **Circulating at TGE** | 50,000,000 (5%) |

## Allocation (1,000,000,000 total)

| Category | % | Tokens | Vesting |
|----------|---|--------|---------|
| Public Sale & Liquidity | 5% | 50,000,000 | Unlocked at TGE (circulating) |
| Ecosystem Incentives | 15% | 150,000,000 | Linear over 48 months |
| Liquidity Mining / LP Rewards | 20% | 200,000,000 | Streamed by emissions schedule |
| Treasury | 25% | 250,000,000 | DAO-governed via TreasuryGovernance |
| Team & Advisors | 20% | 200,000,000 | 12-month cliff, 36-month linear |
| Strategic Partners | 10% | 100,000,000 | 6-month cliff, 24-month linear |
| Insurance Fund Backstop | 5% | 50,000,000 | Locked, unlock by governance only |
| **Total** | **100%** | **1,000,000,000** | — |

All non-circulating tokens are held at the project multisig
([`0x057aB15119212D1308aF26F6491956c82E89b6d5`](https://arbiscan.io/address/0x057aB15119212D1308aF26F6491956c82E89b6d5))
or in on-chain vesting contracts.

## Utility

X963 is the native utility and governance asset of the 963X Protocol:

1. **Governance** — Lock X963 into **veX963** (vote-escrow, Curve-style) to receive non-transferable voting power proportional to lock duration (1 week to 4 years). veX963 holders vote on:
   - New perpetual market listings
   - Maker/taker fee schedules
   - Liquidation parameters
   - Insurance fund deployment
   - Treasury allocation

2. **Real-yield revenue share** — **30% of all protocol trading fees** are streamed weekly in **USDC** to veX963 lockers, proportional to their veX963 balance.

3. **Fee discounts** — Tiered discounts on maker/taker fees based on X963 holdings or veX963 balance.

4. **Buyback & burn** — A portion of protocol revenue funds the on-chain `BuybackEngine`, which buys X963 from the open market and burns it, creating deflationary pressure.

5. **Insurance fund collateral** — A locked allocation backstops liquidation deficits, protecting solvent traders.

## Emission & deflation

- **No inflationary mint** — X963 has a hard cap of 1B; the contract `mint()` is removed after initial allocation.
- **Continuous burn** — `BuybackEngine` burns acquired X963 forever.
- **Net effect** — Deflationary as protocol volume grows.

## ve963X mechanics

```
voting_power(t) = X963_locked × (lock_remaining / max_lock)
max_lock = 4 years (208 weeks)
```

- Decay: linear over time as lock approaches expiry
- Boost: longer locks earn higher fee-share weight
- Transfers: veX963 is non-transferable (soulbound to the locker)

## Fee flow

```
Trader pays fee
        │
        ▼
┌───────────────────┐
│  FeeCollector     │
└─────────┬─────────┘
          │
   ┌──────┼──────────┬─────────────┐
   │      │          │             │
   ▼      ▼          ▼             ▼
30% to   30% to    30% to        10% to
veX963   Treasury  Buyback       Insurance
holders  (DAO)     Engine→burn   Fund
```

## Disclosures

- The X963 contract was deployed at supply 0 and minted into the multisig per the allocation table.
- All vesting is enforced by on-chain contracts; team tokens cannot be moved during cliff.
- Verify allocations live on-chain via Arbiscan.
