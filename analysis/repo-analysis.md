# Repository Analysis: `PP-AR-T/coinnect`

## 1. Repo purpose
`coinnect` is a Rust library that provides REST API clients for multiple crypto exchanges, with:
- exchange-specific/raw API modules, and
- a shared generic API (`ExchangeApi`) to normalize common actions (ticker, orderbook, balances, order placement).

It is explicitly focused on HTTPS REST calls (not streaming/WebSocket/FIX).

## 2. Main folders and components
- `src/lib.rs`: crate entrypoint and exported modules.
- `src/coinnect.rs`: generic factory (`Coinnect`) and credentials trait.
- `src/exchange.rs`: `Exchange` enum and `ExchangeApi` trait.
- `src/types.rs`: shared domain types (`Ticker`, `Orderbook`, `OrderInfo`, `Currency`, `Pair`, etc.).
- `src/error.rs`: central error taxonomy (`error-chain`).
- `src/helpers/`: shared parsing/time/query helpers.
- `src/{bitstamp,kraken,poloniex,bittrex,gdax}/`: per-exchange API, credentials, utils, generic adapter.
- `tests/`: mostly integration tests; many call live public endpoints, private tests behind features.
- `examples/`: usage examples (raw and generic API).
- `.github/workflows/rust.yml`: CI runs `cargo build` and `cargo test`.

## 3. Key files to read first
1. `README.md`
2. `Cargo.toml`
3. `src/exchange.rs`
4. `src/coinnect.rs`
5. `src/types.rs`
6. `src/error.rs`
7. One representative exchange adapter pair:
   - `src/kraken/api.rs`
   - `src/kraken/generic_api.rs`

## 4. Main language/frameworks/dependencies
- Language: Rust (edition 2018).
- HTTP/TLS stack: `hyper 0.10`, `hyper-native-tls 0.3` (legacy synchronous stack).
- Data/parsing: `serde_json`.
- Numeric/time: `bigdecimal`, `chrono`.
- Auth/signing: `hmac`, `sha2`, `data-encoding`.
- Error handling: `error-chain`.
- Utility mapping: `bidir-map`, `lazy_static`.

## 5. How the project appears to run or test
- Build: `cargo build --verbose`
- Test: `cargo test --verbose`
- Optional private-key tests via features in `Cargo.toml` (example: `--features "bitstamp_private_tests"`) with `keys_real.json`.

Observed in this environment: build succeeded; part of tests failed due external network/DNS access to exchange endpoints (live-integration dependency), not due local compile issues.

## 6. Useful concepts worth preserving
- Clear separation between:
  - raw exchange APIs and
  - a normalized exchange-agnostic API trait.
- Centralized domain types (`Ticker`, `Orderbook`, `OrderType`, `Balances`) and exchange-independent interfaces.
- Exchange-specific error translation into a common error vocabulary.
- Pair/currency normalization strategy across heterogeneous exchange naming conventions.
- Credential abstraction and factory-based API construction.

## 7. Code that may be worth reusing
- Architectural pattern:
  - `ExchangeApi` trait and generic adapter design (`src/exchange.rs`, `src/*/generic_api.rs`).
- Domain model shape:
  - shared market/account types in `src/types.rs`.
- Error mapping idea:
  - exchange error string -> common error enum pattern in `src/*/utils.rs`.
- Pair symbol normalization concept:
  - bidirectional per-exchange mapping tables.

## 8. Code that should not be migrated
- Legacy synchronous HTTP/TLS implementation (`hyper 0.10` + blocking request flow).
- Monolithic static `Currency`/`Pair` enums snapshot (large, stale, maintenance-heavy).
- Live-network-coupled tests as core validation strategy.
- Exchange endpoints and assumptions tied to now-changed/renamed venues/APIs.
- Old helper script patterns (e.g., SSL verification bypass in `get_pairs_name.py`).

## 9. Risks and hidden assumptions
- Strong dependence on third-party exchange uptime, endpoint stability, and DNS/network reachability.
- Test reliability heavily depends on live APIs; can produce flaky/non-deterministic CI behavior.
- Hard-coded pair/currency universes become outdated quickly.
- Implicit assumptions around order semantics (e.g., market-order simulation via extreme limit prices).
- Error parsing based on string matching may break when providers change message text.
- Private-test credential file convention (`keys_real.json`) may encourage local secret handling risks if unmanaged.

## 10. Recommended classification
**Partial code reuse**

Reason: the implementation details are dated, but the architecture, abstractions, normalization, and error-translation concepts are strong candidates for transfer into a modern Rust redesign.

## 11. Suggested target mapping into a future Rust repo
- `domain`: `Ticker`, `Orderbook`, `OrderInfo`, `OrderType`, `Balance` model concepts from `src/types.rs`.
- `market-data`: normalized `ticker`/`orderbook` interfaces from `ExchangeApi` + per-venue adapters.
- `strategy`: consume domain abstractions only (no direct migration needed from this repo).
- `execution-sim`: reuse order model semantics; reimplement execution behavior separately.
- `risk`: reuse shared error categories and insufficient-funds/order-size concepts.
- `backtest`: keep as separate module; reuse only normalized data contracts.
- `storage`: no direct reusable storage layer exists; define fresh schemas around normalized domain types.
- `agent-support`: credential abstraction/factory pattern and adapter registration ideas.
- `docs`: migrate architecture notes (generic vs raw API, pair normalization, safety disclaimers).

## 12. One smallest safe next step
Create a minimal RFC/spec in the new Rust repo that freezes **normalized domain contracts** (`Pair`, `Ticker`, `Orderbook`, `OrderRequest`, `OrderResult`, common errors) derived from this project’s concepts, without reusing its transport code.
