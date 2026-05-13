# 963X Public API

> Base URL (REST): `https://api.963x.xyz`
> Base URL (WebSocket): `wss://ws.963x.xyz`
> CoinGecko-compatible endpoints: `https://api.963x.xyz/api/coingecko/*`

## Authentication

Order placement, cancellation, and account queries require an **EIP-712 signature** from the user's wallet. Public market data endpoints are unauthenticated.

```http
POST /order
Content-Type: application/json

{
  "order": { ... },
  "signature": "0x...",
  "nonce": 1715600000123,
  "timestamp": 1715600000123
}
```

## REST — Trading

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/order` | Place a new order |
| `DELETE` | `/order/{orderId}` | Cancel an order |
| `GET` | `/order/{orderId}` | Query a specific order |
| `GET` | `/openOrders?symbol=BTC-PERP` | List open orders |
| `GET` | `/positions` | List open positions |
| `GET` | `/balances` | List balances per asset |
| `GET` | `/markets` | List all tradable markets |
| `GET` | `/fundingRates` | Latest funding rates |

### Order schema

```json
{
  "user_id": "0xabc...",
  "symbol": "BTC-PERP",
  "side": "BUY",
  "type": "LIMIT",
  "price": 67500.00,
  "qty": 0.10,
  "tif": "GTC",
  "reduce_only": false,
  "client_order_id": "client-uuid-..."
}
```

- `side`: `BUY` | `SELL`
- `type`: `LIMIT` | `MARKET` | `STOP` | `TAKE_PROFIT`
- `tif`: `GTC` | `IOC` | `FOK` | `POST_ONLY`

## WebSocket — Market Data

| Channel | Description |
|---------|-------------|
| `orderbook.{symbol}` | L2 orderbook updates |
| `trades.{symbol}` | Trade tape |
| `klines.{symbol}.{interval}` | Candles (1m/5m/15m/1h/4h/1d) |
| `funding.{symbol}` | Funding-rate updates |
| `positions.{user}` | Authenticated position updates |
| `orders.{user}` | Authenticated order updates |
| `system.status` | Health, halts, mode changes |

### Subscribe example

```json
{ "op": "subscribe", "channel": "orderbook.BTC-PERP" }
```

## CoinGecko-compatible endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/coingecko/summary` | All trading pairs summary |
| `GET` | `/api/coingecko/tickers` | 24h tickers |
| `GET` | `/api/coingecko/orderbook?ticker_id=X963_USDC&depth=100` | L2 book |
| `GET` | `/api/coingecko/historical_trades?ticker_id=X963_USDC` | Recent trades |
| `GET` | `/api/coingecko/health` | Health probe |

These endpoints follow the [CoinGecko Decentralized Derivatives Exchange spec](https://docs.coingecko.com/reference/decentralized-exchanges).

## Rate limits

| Tier | Requests/minute | Notes |
|------|-----------------|-------|
| Public | 60 | Per IP |
| Authenticated | 600 | Per user |
| Market-maker (verified) | 6000 | Per API key, on request |

## Error codes

| Code | Meaning |
|------|---------|
| `INSUFFICIENT_MARGIN` | Pre-trade risk check failed |
| `LEVERAGE_EXCEEDED` | Requested leverage > symbol max |
| `MARKET_HALTED` | System in emergency / maintenance |
| `INVALID_SIGNATURE` | EIP-712 signature mismatch |
| `STALE_NONCE` | Nonce / timestamp outside window |
| `RATE_LIMITED` | Too many requests |

## SDK

Official SDKs (TypeScript, Python) — coming soon at `https://github.com/963X-Protocol`.
