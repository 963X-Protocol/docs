# 963X Protocol — Whitepaper (Lite)

**Version 1.0 — May 2026**

## Abstract

963X is a decentralized perpetual futures exchange that delivers centralized-exchange-grade performance — sub-20ms order acks, 125× leverage, deep liquidity — without sacrificing the trust-minimization of on-chain settlement. Built on Arbitrum One, 963X combines an off-chain price-time-priority orderbook with deterministic on-chain custody, settlement, and governance. The native asset, **X963**, captures fee revenue, governs the protocol, and backstops solvency through a vote-escrowed (veX963) model with continuous buy-and-burn.

## 1. Motivation

DEX perpetual venues today force a binary choice: either fully on-chain (low throughput, MEV-exposed) or fully off-chain (custodial risk, opaque). 963X resolves this with a **hybrid architecture**: a high-performance off-chain matching engine for execution, settled deterministically against on-chain vaults users control directly.

## 2. Architecture summary

See [`ARCHITECTURE.md`](../ARCHITECTURE.md) for the full diagram.

- **Off-chain**: Gateway → Risk → Matching → Ledger, all communicating over Kafka with double-entry accounting.
- **On-chain (Arbitrum One)**: PerpDEXVault (custody), PerpDEXSettlement (batched proofs), USDBVaultV2 (BTC/ETH-backed stablecoin), TreasuryGovernance (veX963 votes).
- **Oracles**: Chainlink for mark prices; multi-source fallback.

## 3. Token model

X963 has a **fixed supply of 1 billion**. The full allocation, vesting schedule, and utility breakdown is documented in [`TOKENOMICS.md`](../TOKENOMICS.md). Key properties:

- **Fixed cap, no inflationary mint**
- **30% of trading fees** stream weekly to veX963 lockers in USDC (real yield)
- **Continuous buyback & burn** funded from protocol revenue
- **Vote-escrow governance** (1 week to 4 year locks)

## 4. Risk framework

- **Initial margin** required pre-trade; orders rejected if insufficient.
- **Maintenance margin** triggers liquidation; partial close first to minimize market impact.
- **Insurance fund** absorbs liquidation deficits; ADL waterfall as last resort.
- **Mark price** decoupled from last trade to defeat wash-trade liquidations.

## 5. Governance

ve963X holders propose and vote on:

- Market listings & delistings
- Fee schedules
- Risk parameters (max leverage, IM/MM ratios)
- Treasury allocations
- Insurance fund deployment

Proposals execute via on-chain timelock through `TreasuryGovernance`, with the multisig as a final safety net during the bootstrap phase.

## 6. Roadmap

| Phase | Status | Highlights |
|-------|--------|------------|
| Mainnet alpha | ✅ Live | Arbitrum One, BTC-PERP, ETH-PERP |
| Multi-asset expansion | 🚧 In progress | 50+ crypto perps, FX, prediction markets |
| Cross-chain settlement | 📅 Planned | Optimism, Base, BNB Chain |
| Mobile app | 📅 Planned | iOS / Android |
| Full DAO transition | 📅 Planned | Multisig → 100% veX963 governance |

## 7. References

- Smart contracts: https://github.com/963X-Protocol/contracts
- API docs: [`API.md`](../API.md)
- Deployed addresses: [`DEPLOYED_ADDRESSES.md`](../DEPLOYED_ADDRESSES.md)
- Security policy: [`SECURITY.md`](../SECURITY.md)
- DEX: https://963x.xyz

---

*This document is a public-facing summary. The full technical specification — including formal proofs, sequence diagrams, and economic simulations — is maintained internally and selected sections are open-sourced as the protocol decentralizes.*
