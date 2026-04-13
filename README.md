<h1 align="center">Centralized Exchange Engine</h1>
<p align="center"><strong>High-Performance Order Matching Engine in Rust</strong></p>

<p align="center">
  <img src="https://img.shields.io/badge/Rust-1.87+-DEA584?style=flat-square&logo=rust" />
  <img src="https://img.shields.io/badge/Axum-0.7-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/Tokio-async-purple?style=flat-square" />
  <img src="https://img.shields.io/badge/REST_API-✓-00D18C?style=flat-square" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" />
</p>

<p align="center">
  A modular centralized exchange engine featuring price-time priority order matching, multi-market support, and a RESTful API interface. Supports limit and market orders with O(log n) matching operations.
</p>

---

## The Problem

Building an exchange requires solving several hard problems simultaneously: maintaining a fair and deterministic order matching algorithm, supporting multiple trading pairs without contention, and exposing it all through a performant API that can handle concurrent requests. Most implementations either sacrifice correctness for speed or are monolithic and difficult to extend.

## The Solution

This engine splits the problem into three cleanly separated layers — each with a single responsibility and well-defined interface:

| Layer | Crate | Responsibility |
|:---:|---|---|
| **1** | `orderbook` | Price-time priority matching, limit/market order execution, O(log n) insert/cancel |
| **2** | `trading_engine` | Multi-market management, trading pair validation, unified error handling |
| **3** | `server` | Axum-based REST API, async request handling, JSON serialization |

Requests flow down through the layers and responses flow back up. Each layer only knows about the one directly below it.

---

## Features

- **Price-time priority matching** — orders at the same price level are filled in the order they were placed, ensuring fairness
- **Limit and market orders** — full support for both order types with partial fill handling
- **Multi-market support** — create and manage multiple trading pairs (BTC/USD, ETH/USD, etc.) concurrently
- **Order lifecycle management** — place, modify, and cancel orders with consistent state guarantees
- **Market depth** — query the full orderbook state for any trading pair, including mid-price calculation
- **Async REST API** — Axum-based HTTP server with Tokio runtime for non-blocking request handling
- **Modular architecture** — three independent crates with clear boundaries, testable in isolation

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        HTTP Client                           │
│                                                              │
│   ┌─────────────┐  ┌──────────────┐  ┌───────────────────┐  │
│   │ Place Order  │  │ Modify/Cancel│  │  Query Depth /    │  │
│   │ (Limit/Mkt)  │  │   Order      │  │  Mid Price        │  │
│   └──────┬───────┘  └──────┬───────┘  └─────────┬─────────┘  │
└──────────┼─────────────────┼────────────────────┼────────────┘
           │           REST API                    │
┌──────────┴─────────────────┴────────────────────┴────────────┐
│               Layer 3: HTTP Server (Axum)                    │
│                                                              │
│  Validates requests, acquires engine lock, routes to engine  │
│  Serializes responses back to JSON                           │
└──────────────────────────┬───────────────────────────────────┘
                           │
┌──────────────────────────┴───────────────────────────────────┐
│              Layer 2: Trading Engine                         │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Market Registry                                        │ │
│  │  BTC/USD ──→ Orderbook    ETH/USD ──→ Orderbook         │ │
│  │  SOL/USD ──→ Orderbook    ...                           │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  Validates market exists, routes to correct orderbook        │
└──────────────────────────┬───────────────────────────────────┘
                           │
┌──────────────────────────┴───────────────────────────────────┐
│              Layer 1: Orderbook                              │
│                                                              │
│  ┌────────────────┐              ┌────────────────┐          │
│  │   Bids (Buy)   │              │  Asks (Sell)   │          │
│  │  ┌───────────┐ │              │ ┌───────────┐  │          │
│  │  │ 50,100.00 │ │   Mid Price  │ │ 50,200.00 │  │          │
│  │  │ 50,050.00 │ │◄────────────►│ │ 50,250.00 │  │          │
│  │  │ 50,000.00 │ │              │ │ 50,300.00 │  │          │
│  │  └───────────┘ │              │ └───────────┘  │          │
│  │  Price-Time    │              │  Price-Time    │          │
│  │  Priority      │              │  Priority      │          │
│  └────────────────┘              └────────────────┘          │
│                                                              │
│  O(log n) insert/cancel, partial fills, trade execution      │
└──────────────────────────────────────────────────────────────┘
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/v1/create-market` | Create a new trading pair |
| `GET` | `/api/v1/get-market` | List all active markets |
| `POST` | `/api/v1/limit-order` | Place a limit order |
| `POST` | `/api/v1/market-order` | Place a market order |
| `POST` | `/api/v1/modify-order` | Modify an existing order |
| `DELETE` | `/api/v1/delete-order` | Cancel an order |
| `GET` | `/api/v1/get-order` | Get order details |
| `GET` | `/api/v1/depth` | Get full orderbook depth |
| `GET` | `/api/v1/mid-price` | Get current mid price |

---

## Quick Start

### Prerequisites

- Rust 1.87+
- Cargo

### 1. Clone and build

```bash
git clone https://github.com/prranavv/rust-trading-system.git
cd rust-trading-system

cargo build --release
```

### 2. Run the server

```bash
cargo run --release
```

The server starts on `http://0.0.0.0:8000`.

### 3. Test

```bash
cargo test
```

Run tests for a specific crate:

```bash
cargo test -p orderbook
cargo test -p trading_engine
cargo test -p server
```

---

## Example: Complete Trading Flow

```bash
# 1. Create a market
curl -X POST http://localhost:8000/api/v1/create-market \
  -H "Content-Type: application/json" \
  -d '{"trading_pair": {"base": "BTC", "quote": "USD"}}'

# 2. Add liquidity — place a buy limit order
curl -X POST http://localhost:8000/api/v1/limit-order \
  -H "Content-Type: application/json" \
  -d '{
    "trading_pair": {"base": "BTC", "quote": "USD"},
    "order": {
      "price": "49900",
      "quantity": "1.0",
      "side": "Bids",
      "user_id": 1
    }
  }'

# 3. Execute against it — place a sell market order
curl -X POST http://localhost:8000/api/v1/market-order \
  -H "Content-Type: application/json" \
  -d '{
    "trading_pair": {"base": "BTC", "quote": "USD"},
    "order": {
      "quantity": "0.5",
      "side": "Asks",
      "user_id": 2
    }
  }'

# 4. Check market depth
curl -X GET http://localhost:8000/api/v1/depth \
  -H "Content-Type: application/json" \
  -d '{"trading_pair": {"base": "BTC", "quote": "USD"}}'
```

---

## Project Structure

```
rust-trading-system/
├── orderbook/                  # Layer 1: Core matching engine
│   ├── src/
│   │   └── lib.rs              # Orderbook, price levels, matching logic
│   └── Cargo.toml
│
├── trading_engine/             # Layer 2: Multi-market management
│   ├── src/
│   │   └── lib.rs              # Market registry, validation, routing
│   └── Cargo.toml
│
├── server/                     # Layer 3: REST API
│   ├── src/
│   │   └── main.rs             # Axum routes, handlers, JSON API
│   └── Cargo.toml
│
├── Cargo.toml                  # Workspace root
├── README.md
└── LICENSE
```

---

## Tech Stack

| Component | Technology | Purpose |
|---|---|---|
| **Language** | Rust | Memory-safe systems programming with zero-cost abstractions |
| **HTTP Framework** | Axum | Ergonomic async web framework built on Tokio and Tower |
| **Async Runtime** | Tokio | Non-blocking I/O for concurrent request handling |
| **Serialization** | Serde + serde_json | JSON request/response serialization |
| **Build** | Cargo workspaces | Multi-crate project with shared dependencies |

---

## FAQ's

**"Why Rust for an exchange engine?"**
> Correctness and performance are non-negotiable for financial systems. Rust's ownership model eliminates data races at compile time, which matters when multiple API requests are contending for the same orderbook. The zero-cost abstractions mean you get the safety of a high-level language with the performance of C.

**"Why a centralized engine instead of on-chain?"**
> On-chain orderbooks face fundamental throughput and latency constraints. A centralized engine can match orders in microseconds with deterministic behavior, which is the baseline expectation for any serious trading system. The tradeoff is custody and trust — but the matching logic itself is where the complexity lives.

**"Why price-time priority?"**
> It's the fairest and most widely used matching algorithm. Orders at the same price are filled in the order they arrived, which means early participants are rewarded and there's no way to front-run at the same price level. Every major exchange (NYSE, CME, Binance) uses some variant of this.

**"Why separate the orderbook, engine, and server into different crates?"**
> Each layer has a different rate of change and different testing requirements. The orderbook is pure logic with no I/O — it can be tested exhaustively with unit tests. The trading engine adds market management but still no I/O. The server is the only layer that touches the network. Separating them means you can swap out the API layer (say, switch to WebSockets or gRPC) without touching the matching logic.

**"How does it handle concurrent requests?"**
> The trading engine is behind a shared lock (Arc<Mutex>) in the Axum state. Each request acquires the lock, performs the operation, and releases it. This is simple and correct for moderate throughput. For higher performance, you'd move to a single-threaded event loop with an MPSC channel — removing lock contention entirely.

---

## Disclaimer

This is a portfolio project demonstrating exchange engine architecture and order matching logic in Rust. It is not intended for production use with real funds. There is no authentication, no persistence layer, no risk management, and no regulatory compliance. Always conduct a thorough security audit before deploying any financial system.

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

<p align="center">
  <sub>Built by <a href="https://github.com/prranavv">Pranav</a></sub>
</p>