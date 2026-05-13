# Security

## Threat model

963X is a non-custodial protocol. The off-chain matching engine **never holds user keys** and **cannot withdraw user funds**. All settlement and custody operations are enforced on-chain by audited contracts.

## Multisig & access control

- **Production multisig**: Gnosis Safe **3-of-5**, [`0x057aB15119212D1308aF26F6491956c82E89b6d5`](https://arbiscan.io/address/0x057aB15119212D1308aF26F6491956c82E89b6d5)
- All admin roles (`UPGRADER_ROLE`, `PARAM_ADMIN_ROLE`, `EMERGENCY_ROLE`) granted **only** to the multisig.
- Sensitive operations (upgrades, parameter changes >X%, treasury transfers) gated by an on-chain timelock.

## Smart-contract safeguards

- OpenZeppelin v5 base contracts (audited by Trail of Bits, OpenZeppelin)
- `ReentrancyGuard` on every state-mutating external function
- `SafeERC20` for all token transfers
- Numeric: `NUMERIC(38,18)` on the off-chain ledger; on-chain uses uint256 with explicit overflow checks (Solidity ≥0.8)
- Pre-trade risk checks: cannot place an order without sufficient initial margin
- Mark price (Chainlink-anchored) ≠ last trade price → resistant to wash-trade liquidations

## Operational safeguards

- **Circuit breakers** auto-trigger on:
  - Oracle staleness > N seconds
  - Price deviation > 5% in a sliding window
  - Liquidation queue overflow
- **Emergency halt** switches the protocol to *reduce-only* mode; new positions blocked.
- Off-chain engines are restartable in < 30s via Kafka replay.

## Bug bounty

We welcome responsible disclosure of vulnerabilities.

- **Email**: `security@963x.xyz` (PGP key on request)
- **Scope**: smart contracts in [`963X-Protocol/contracts`](https://github.com/963X-Protocol/contracts), production APIs at `api.963x.xyz`, and the DEX frontend at `https://963x.xyz`.
- **Rewards** (USDC, paid by multisig):

| Severity | Reward |
|----------|--------|
| Critical (loss of funds) | up to $250,000 |
| High (privilege escalation, oracle manipulation) | up to $50,000 |
| Medium (DoS of core flows) | up to $10,000 |
| Low (informational) | up to $1,000 |

Out of scope: known issues, third-party RPC outages, social engineering, automated scanner reports without PoC.

## Audits

Audit reports will be published to this repository under `audits/` as they complete.
