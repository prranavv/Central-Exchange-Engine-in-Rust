# Centralized Exchange Engine in Rust

A high-performance, modular centralized exchange (CEX) engine built in Rust, featuring a complete order matching engine, multi-market support, and a RESTful API interface. Implements core exchange functionality including order placement, price-time priority matching, and trade execution.

## 🏗️ Project Structure
```
.
├── orderbook/          # Core order matching engine
├── trading_engine/     # Multi-market management layer
└── server/            # HTTP API server
```

### Component Overview

| Component | Description | Key Features |
|-----------|-------------|--------------|
| **Orderbook** | Low-level order matching engine | Price-time priority matching, Limit/Market orders, O(log n) operations |
| **Trading Engine** | Market management layer | Multiple trading pairs, Market validation, Unified error handling |
| **Server** | REST API interface | Axum-based HTTP server, Async request handling, JSON API |

## 🚀 Quick Start

### Prerequisites

- Rust 1.87+
- Cargo

### Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/rust-trading-system.git
cd rust-trading-system
```

2. Build all components:
```bash
cargo build --release
```

3. Run the server:
```bash
cargo run --release
```

The server will start on `http://0.0.0.0:8000`

### Basic Usage Example

```bash
# Create a new market
curl -X POST http://localhost:8000/api/v1/create-market \
  -H "Content-Type: application/json" \
  -d '{"trading_pair": {"base": "BTC", "quote": "USD"}}'

# Place a limit order
curl -X POST http://localhost:8000/api/v1/limit-order \
  -H "Content-Type: application/json" \
  -d '{
    "trading_pair": {"base": "BTC", "quote": "USD"},
    "order": {
      "price": "50000.00",
      "quantity": "0.5",
      "side": "Bids",
      "user_id": 1
    }
  }'

# Get market depth
curl -X GET http://localhost:8000/api/v1/depth \
  -H "Content-Type: application/json" \
  -d '{"trading_pair": {"base": "BTC", "quote": "USD"}}'
```

### Data Flow

1. **Client Request** → HTTP Server receives and validates request
2. **Server** → Acquires lock on Trading Engine
3. **Trading Engine** → Validates market exists and routes to correct orderbook
4. **Orderbook** → Executes order matching/operations
5. **Response** → Flows back through the layers to client

## 📡 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/create-market` | Create new trading pair |
| GET | `/api/v1/get-market` | List all markets |
| POST | `/api/v1/limit-order` | Place limit order |
| POST | `/api/v1/market-order` | Place market order |
| POST | `/api/v1/modify-order` | Modify existing order |
| DELETE | `/api/v1/delete-order` | Cancel order |
| GET | `/api/v1/get-order` | Get order details |
| GET | `/api/v1/depth` | Get market depth |
| GET | `/api/v1/mid-price` | Get mid price |

## 🧪 Testing

Run all tests:
```bash
cargo test
```

Run tests for specific component:
```bash
cargo test -p orderbook
cargo test -p trading_engine
cargo test -p server
```

Run with verbose output:
```bash
cargo test -- --nocapture
```

## 🔧 Configuration

### Server Port
Edit `server/src/main.rs`:
```rust
let listener = tokio::net::TcpListener::bind("0.0.0.0:8000").await.unwrap();
```

## 📝 Example: Complete Trading Flow

```rust
// 1. Start the server
// cargo run --release

// 2. Create a market (via HTTP)
POST /api/v1/create-market
{"trading_pair": {"base": "BTC", "quote": "USD"}}

// 3. Add liquidity (limit orders)
POST /api/v1/limit-order
{
  "trading_pair": {"base": "BTC", "quote": "USD"},
  "order": {
    "price": "49900",
    "quantity": "1.0",
    "side": "Bids",
    "user_id": 1
  }
}

// 4. Execute market order
POST /api/v1/market-order
{
  "trading_pair": {"base": "BTC", "quote": "USD"},
  "order": {
    "quantity": "0.5",
    "side": "Asks",
    "user_id": 2
  }
}

// 5. Check market depth
GET /api/v1/depth
{"trading_pair": {"base": "BTC", "quote": "USD"}}
```

## 🛠️ Troubleshooting

| Issue | Solution |
|-------|----------|
| Port already in use | Change port in `main.rs` or kill process on port 8000 |
| Compilation errors | Ensure Rust 1.70+ and run `cargo update` |
| Market not found | Create market first with `/api/v1/create-market` |
| Order matching issues | Check orderbook has liquidity on opposite side |

## 📚 Documentation

- [Orderbook Documentation](./orderbook/README.md)
- [Trading Engine Documentation](./trading_engine/README.md)
- [Server API Documentation](./server/README.md)
- [Rust API Docs](https://docs.rs): Run `cargo doc --open`

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.
