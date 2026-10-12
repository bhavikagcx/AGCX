# AGCX Public API

**Arab Global Crypto Exchange (AGCX) – Market Data API for Spot Markets & Perpetual Futures**

| | |
|---|---|
| **Version** | 1.0 |
| **Last updated** | 12 October 2026 |
| **Base URL** | `https://backend.arabglobal.ae` |
| **Website** | [arabglobal.ae](https://arabglobal.ae) |

---

## Table of Contents

1. [Overview](#1-overview)
2. [Quick Start](#2-quick-start)
3. [Endpoint Summary](#3-endpoint-summary)
4. [Conventions](#4-conventions)
5. [Spot Endpoints](#5-spot-endpoints)
   - [5.1 Spot Tickers](#51-spot-tickers)
   - [5.2 Spot Order Book](#52-spot-order-book)
   - [5.3 Spot Historical Trades](#53-spot-historical-trades)
6. [Futures Endpoints](#6-futures-endpoints)
   - [6.1 Futures Contracts](#61-futures-contracts)
   - [6.2 Futures Order Book](#62-futures-order-book)
7. [Market Type Identification](#7-market-type-identification)
8. [Links](#8-links)

---

## 1. Overview

AGCX provides public, read-only HTTP `GET` endpoints for market data. All responses are returned as JSON. The API allows external market-data services and aggregators to retrieve:

- Spot ticker summaries (last price and 24-hour volume)
- Spot order books
- Spot trade history
- Perpetual futures contract details
- Perpetual futures order books

> **Scope:** These endpoints expose public market data only. They do not provide account access, order placement, deposits, withdrawals, or any other authenticated trading action.

---

## 2. Quick Start

No API key or registration is required.

```bash
# All active spot markets
curl "https://backend.arabglobal.ae/api/v1/agcx/spot/tickers"

# Spot order book
curl "https://backend.arabglobal.ae/api/v1/agcx/spot/orderbook?ticker_id=BTC_USDT&depth=100"

# Spot historical trades (buy side)
curl "https://backend.arabglobal.ae/api/v1/agcx/spot/historical_trades?ticker_id=BTC_USDT&type=buy"

# Perpetual futures contracts
curl "https://backend.arabglobal.ae/api/v1/agcx/futures/contracts"

# Futures order book
curl "https://backend.arabglobal.ae/api/v1/agcx/futures/orderbook?ticker_id=BTCUSDT-PERP&depth=100"
```

---

## 3. Endpoint Summary

| Endpoint | Method | Path | Description |
|---|---|---|---|
| Spot tickers | `GET` | `/api/v1/agcx/spot/tickers` | Ticker summary (latest price, 24h volume) for every active spot market |
| Spot order book | `GET` | `/api/v1/agcx/spot/orderbook` | Order book for a spot market |
| Spot historical trades | `GET` | `/api/v1/agcx/spot/historical_trades` | Historical trades by market and side |
| Futures contracts | `GET` | `/api/v1/agcx/futures/contracts` | Listed perpetual futures contracts and metadata |
| Futures order book | `GET` | `/api/v1/agcx/futures/orderbook` | Order book for a perpetual futures contract |

All paths are relative to `https://backend.arabglobal.ae`.

---

## 4. Conventions

| Item | Value |
|---|---|
| HTTP method | `GET` |
| Authentication | None (public, read-only) |
| Response format | `application/json` |
| Base URL | `https://backend.arabglobal.ae` |
| Spot market identifier | `BASE_QUOTE`, e.g. `BTC_USDT` |
| Futures contract identifier | e.g. `BTCUSDT-PERP` (use the `ticker_id` returned by the contracts endpoint) |
| Prices and sizes | Decimal strings |

---

## 5. Spot Endpoints

### 5.1 Spot Tickers

Returns one ticker summary per active spot market, including the latest traded price and 24-hour volume.

```
GET https://backend.arabglobal.ae/api/v1/agcx/spot/tickers
```

**Parameters:** none

**Response fields**

| Field | Description |
|---|---|
| `ticker_id` | Market identifier, e.g. `BTC_USDT` |
| `base_currency` | Base asset symbol, e.g. `BTC` |
| `target_currency` | Quote asset symbol, e.g. `USDT` |
| `last_price` | Latest traded price |
| `base_volume` | 24-hour traded volume in the base asset |
| `target_volume` | 24-hour traded volume in the quote asset |
| `bid` | Best available bid price |
| `ask` | Best available ask price |
| `high` | 24-hour high price |
| `low` | 24-hour low price |

---

### 5.2 Spot Order Book

Returns the current order book for the requested spot market.

```
GET https://backend.arabglobal.ae/api/v1/agcx/spot/orderbook?ticker_id=BTC_USDT&depth=100
```

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `ticker_id` | Yes | Spot market identifier, e.g. `BTC_USDT` |
| `depth` | No | Total number of price levels, split evenly between bids and asks |

**Response structure**

```json
{
  "ticker_id": "BTC_USDT",
  "timestamp": "<unix-milliseconds>",
  "bids": [["<price>", "<size>"]],
  "asks": [["<price>", "<size>"]]
}
```

| Field | Description |
|---|---|
| `ticker_id` | Market identifier |
| `timestamp` | Snapshot time, Unix milliseconds |
| `bids` | Array of `[price, size]`, ordered highest to lowest price |
| `asks` | Array of `[price, size]`, ordered lowest to highest price |

---

### 5.3 Spot Historical Trades

Returns historical trades for a spot market, filtered by trade side. Query each side (`buy`, `sell`) separately.

```
GET https://backend.arabglobal.ae/api/v1/agcx/spot/historical_trades?ticker_id=BTC_USDT&type=buy
```

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `ticker_id` | Yes | Spot market identifier, e.g. `BTC_USDT` |
| `type` | Yes | Trade side: `buy` or `sell` |
| `limit` | No | Maximum number of trades to return |
| `start_time` | No | Start of time range, Unix seconds |
| `end_time` | No | End of time range, Unix seconds |

**Response structure**

```json
{
  "buy": [
    {
      "trade_id": 0,
      "price": "<price>",
      "base_volume": "<amount>",
      "target_volume": "<amount>",
      "trade_timestamp": "<unix-seconds>",
      "type": "buy"
    }
  ],
  "sell": []
}
```

| Field | Description |
|---|---|
| `trade_id` | Unique trade identifier (integer) |
| `price` | Trade price |
| `base_volume` | Trade size in the base asset |
| `target_volume` | Trade size in the quote asset |
| `trade_timestamp` | Trade time, Unix seconds |
| `type` | Trade side: `buy` or `sell` |

---

## 6. Futures Endpoints

AGCX lists USD-margined **perpetual** futures contracts.

### 6.1 Futures Contracts

Returns all listed perpetual futures contracts and their metadata.

```
GET https://backend.arabglobal.ae/api/v1/agcx/futures/contracts
```

**Parameters:** none

**Response fields**

| Field | Description |
|---|---|
| `ticker_id` | Contract identifier, e.g. `BTCUSDT-PERP` |
| `base_currency` | Base asset symbol |
| `target_currency` | Quote asset symbol |
| `last_price` | Latest traded price |
| `high` / `low` | 24-hour high and low prices |
| `base_volume` | 24-hour volume in the base asset |
| `target_volume` | 24-hour volume in the quote asset |
| `bid` / `ask` | Best bid and ask prices |
| `product_type` | Contract classification (`Perpetual`) |
| `contract_type` | Contract type (`Vanilla`) |
| `contract_price` | Contract price |
| `contract_price_currency` | Quote currency of the contract price |
| `index_name` | Name of the reference index |
| `index_currency` | Quote currency of the index |
| `funding_rate` | Current funding rate |
| `open_interest` | Open interest |

---

### 6.2 Futures Order Book

Returns the current order book for the specified perpetual futures contract.

```
GET https://backend.arabglobal.ae/api/v1/agcx/futures/orderbook?ticker_id=BTCUSDT-PERP&depth=100
```

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `ticker_id` | Yes | Contract identifier, e.g. `BTCUSDT-PERP` |
| `depth` | No | Total number of price levels, split evenly between bids and asks |

**Response structure**

```json
{
  "ticker_id": "BTCUSDT-PERP",
  "timestamp": "<unix-milliseconds>",
  "bids": [["<price>", "<size>"]],
  "asks": [["<price>", "<size>"]]
}
```

The structure matches the spot order book: each price level is `[price, size]`, with bids ordered highest to lowest and asks lowest to highest.

---

## 7. Market Type Identification

Spot and derivatives markets are served from separate, clearly distinguished endpoint groups:

| Market type | Endpoint prefix | Identifier format |
|---|---|---|
| Spot | `/api/v1/agcx/spot/` | `BASE_QUOTE` (e.g. `BTC_USDT`) |
| Perpetual futures | `/api/v1/agcx/futures/` | Contract ID (e.g. `BTCUSDT-PERP`) |

Futures contracts additionally carry `product_type` (`Perpetual`) and `contract_type` in the `/futures/contracts` response.

---

## 8. Links

- **Website:** <https://arabglobal.ae>
- **API host:** <https://backend.arabglobal.ae>

---

*© Arab Global Crypto Exchange. Market data is provided for informational purposes.*
