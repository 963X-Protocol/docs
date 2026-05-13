# 963X Protocol — Architecture

## Overview

963X is a decentralized perpetual futures exchange combining the speed of a **central-limit orderbook (CLOB)** with the trust-minimization of **on-chain settlement** on Arbitrum One. The protocol supports up to **125× leverage** across:

- Crypto perpetuals (BTC, ETH, SOL, ARB, +50 majors)
- FX perpetuals (EUR/USD, GBP/USD, USD/JPY, …)
- Prediction markets (binary outcomes, settled by oracle)

## High-level diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                     User (web / mobile / API)                     │
└─────────────────────────────┬────────────────────────────────────┘
                              │ EIP-712 signed orders
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  Off-chain Engine (Rust + Node.js)                               │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐ │
│  │  Gateway   │→ │  Risk      │→ │  Matching  │→ │  Ledger    │ │
│  │  (REST/WS) │  │  Engine    │  │  Engine    │  │  Engine    │ │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘ │
│         │                                              │         │
│         └────────────── Kafka event bus ───────────────┘         │
└─────────────────────────────┬────────────────────────────────────┘
                              │ Settlement batches
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│  On-chain (Arbitrum One)                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ PerpDEXVault │  │ Settlement   │  │ Funding      │           │
│  │ (custody)    │  │ (batch exec) │  │ Engine       │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ USDB Vault   │  │ Treasury     │  │ Buyback      │           │
│  │ (BTC/ETH→$)  │  │ Governance   │  │ Engine       │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                              │                                   │
│  ┌───────────────────────────▼──────────────────────────┐        │
│  │ Chainlink Oracles (BTC, ETH, FX, …)                  │        │
│  └──────────────────────────────────────────────────────┘        │
└──────────────────────────────────────────────────────────────────┘
```

## Core invariants

1. **Non-custodial** — User funds live in `PerpDEXVault`. The off-chain engine has zero withdraw authority.
2. **Deterministic settlement** — Every trade is replayable from the on-chain event log.
3. **Pre-trade risk checks** — Margin, leverage, position-size limits are enforced *before* an order enters the book.
4. **Double-entry ledger** — Every balance mutation produces ≥4 immutable ledger entries (debit/credit pairs + fees).
5. **Mark price ≠ last trade** — Liquidations are triggered by a Chainlink-anchored mark price, not a manipulable last-trade.

## Service breakdown

### Gateway
- REST: order placement, cancellation, queries
- WebSocket: orderbook, trades, positions, system status
- Authn: EIP-712 signed payloads + nonce/timestamp window

### Risk Engine
- Pre-trade: required initial margin, max leverage, max open orders
- Post-trade: margin-ratio monitor → liquidation triggers
- Circuit breakers: deviation, oracle staleness, queue depth

### Matching Engine
- Price-time priority CLOB
- Order types: LIMIT, MARKET, STOP, TAKE_PROFIT
- TIF: GTC, IOC, FOK, POST_ONLY

### Ledger Engine
- Append-only `ledger_entries` table (NUMERIC(38,18))
- Balance = `SUM(entries WHERE user=…, asset=…)`
- Idempotent on Kafka offsets

### Funding Engine
- 8-hour funding cadence (or 1h for high-vol markets)
- Rate = clamp(premium + interest, ±max_rate)
- Settled atomically against PnL ledger

### Settlement (on-chain)
- Batched proofs posted to `PerpDEXSettlement` contract
- Withdrawals require on-chain approval, executed by user

## Performance targets (SLO)

| Metric | Target |
|--------|--------|
| Place-order ack | < 20 ms |
| Match execution | < 10 ms |
| WS market-data delay | < 50 ms |
| Uptime | 99.99% |
| Engine restart | < 30 s |
| Full replay recovery | < 5 min |

## Failure modes & mitigations

| Failure | Mitigation |
|---------|------------|
| Matching engine crash | Hot replica + Kafka replay (< 30s RTO) |
| Oracle stale / off-peg | Auto-pause new orders; reduce-only mode |
| Liquidation cascade | Insurance fund → ADL waterfall |
| Bridge / RPC outage | Graceful degradation; settlement queues |
| Multisig key compromise | 3-of-5 threshold + timelock on critical ops |
