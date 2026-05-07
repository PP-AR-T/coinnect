# Repository analysis: `PP-AR-T/coinnect`

## 1. Repo purpose
`coinnect` is a Rust library for accessing multiple cryptocurrency exchange REST APIs through both exchange-specific clients and a generic normalized interface. It is focused on HTTPS request/response access, not streaming or websocket connectivity.

Because the project is explicitly oriented around exchange access, balances, orderbooks, and order placement, it should be treated as a reference artifact only for migration planning. It should **not** be directly reused for any future live-trading integration work.

## 2. Main folders and components
- `src/lib.rs` — crate entry point and public module exports.
- `src/coinnect.rs` — generic `Coinnect` factory and shared credentials trait.
- `src/exchange.rs` — `Exchange` enum and normalized `ExchangeApi` trait.
- `src/types.rs` — shared domain-ish types like `Ticker`, `Orderbook`, `OrderInfo`, `OrderType`, `Currency`, and `Pair`.
- `src/error.rs` — centralized error model.
- `src/helpers/` — shared helper utilities.
- `src/bitstamp/`, `src/bittrex/`, `src/kraken/`, `src/poloniex/`, `src/gdax/` — exchange-specific implementations.
- `examples/` — example usage, including generic API usage and a trading example.
- `tests/` — integration-style tests, many of which depend on live exchange APIs.
- `get_pairs_name.py` — auxiliary script related to pair discovery.
- `README.md` and `TODO.md` — project overview and backlog.

## 3. Key files to read first
1. `README.md`
2. `Cargo.toml`
3. `src/lib.rs`
4. `src/exchange.rs`
5. `src/coinnect.rs`
6. `src/types.rs`
7. `src/error.rs`
8. `examples/generic_api.rs`
9. `tests/coinnect.rs`
10. One representative exchange folder such as:
   - `src/kraken/api.rs`
   - `src/kraken/generic_api.rs`
   - `src/kraken/credentials.rs`
   - `src/kraken/utils.rs`

## 4. Main language/frameworks/dependencies
- Language: Rust 2018 edition.
- Networking: `hyper = "0.10.10"`, `hyper-native-tls = "0.3"`.
- Serialization/data handling: `serde_json`.
- Numeric/time: `bigdecimal`, `chrono`.
- Crypto/signing: `sha2`, `hmac`, `data-encoding`.
- Error handling: `error-chain`.
- Utility crates: `lazy_static`, `bidir-map`.
- CI/test hints: legacy Travis CI via `.travis.yml`.

This is an older Rust stack with legacy blocking HTTP patterns and outdated exchange assumptions.

## 5. How the project appears to run or test
From the repository metadata and files:
- Build: `cargo build --verbose`
- Test: `cargo test --verbose`
- Formatting/linting in CI: `cargo fmt` and `cargo clippy` on nightly in `.travis.yml`
- Optional private tests are enabled with features such as:
  - `bitstamp_private_tests`
  - `kraken_private_tests`
  - `poloniex_private_tests`
  - `bittrex_private_tests`

The README states that private tests expect a root-level `keys_real.json` file with exchange API credentials. Tests appear to call real external endpoints, so they are integration tests against live services rather than isolated deterministic tests.

## 6. Useful concepts worth preserving
- A normalized exchange-agnostic trait (`ExchangeApi`) for common operations.
- A factory pattern (`Coinnect::new`, `Coinnect::new_from_file`) to construct adapter instances.
- Shared canonical market/domain types for ticker, orderbook, balances, and order placement results.
- Translation from exchange-specific formats into a common interface.
- Pair and currency normalization across heterogeneous exchange symbol naming.
- Separation between raw exchange APIs and higher-level generic access.
- Explicit error normalization into common categories.

These ideas are worth preserving as concepts, not as direct implementation carryover.

## 7. Code that may be worth reusing
- Conceptual structure from `src/exchange.rs` and `src/coinnect.rs`.
- Shapes of normalized data models from `src/types.rs`.
- Error taxonomy approach from `src/error.rs` and exchange utility modules.
- Credential abstraction ideas from per-exchange `credentials.rs` files.
- Mapping-table patterns for symbol normalization.

Recommended reuse level: **selective conceptual reuse** or small non-sensitive type definitions after careful review, not wholesale code copy.

## 8. Code that should not be migrated
- Any exchange-specific REST client implementations under:
  - `src/bitstamp/`
  - `src/bittrex/`
  - `src/kraken/`
  - `src/poloniex/`
  - `src/gdax/`
- Trading example logic in `examples/kraken_trading.rs`.
- Any code path for placing real orders or fetching live balances.
- Legacy blocking HTTP stack and old TLS/client code.
- Large static `Currency` and `Pair` enums as-is, because they are stale and maintenance-heavy.
- Test patterns that depend on real credentials or live APIs.
- `get_pairs_name.py` if it embeds unsafe networking assumptions or old data collection shortcuts.

## 9. Risks and hidden assumptions
- **Credential handling risk:** README and tests document a `keys_real.json` file containing real API keys, secrets, and customer IDs for private tests.
- **Live-trading assumptions:** the codebase includes order placement abstractions and a trading example, which makes direct reuse unsafe.
- **External dependency risk:** tests and runtime behavior depend on third-party exchange uptime, network access, and unchanged remote APIs.
- **Staleness risk:** exchange APIs, exchange names, and asset pairs have likely changed significantly since this code was written.
- **Model staleness:** static pair/currency enumerations represent a frozen snapshot, not a durable source of truth.
- **Operational safety risk:** direct balance and order methods can create accidental real-world financial impact if reused carelessly.
- **Testing risk:** integration-heavy tests are flaky by nature and unsuitable as a migration baseline.
- **Legacy ecosystem risk:** very old dependencies imply likely modernization work even for benign reuse.

## 10. Recommended classification
**Concept rewrite**

Reason: the repository contains useful architectural ideas and normalized abstractions, but it is tightly tied to legacy exchange integrations, credential-driven private tests, and live-order concepts. That makes it inappropriate for direct migration and safer to treat as a design reference for a fresh implementation.

## 11. Suggested target mapping into a future Rust repo
| Future area | Suggested mapping from this repo |
|---|---|
| `domain` | Re-specify normalized entities inspired by `Ticker`, `Orderbook`, `OrderInfo`, `OrderType`, balance concepts, and symbol identifiers. |
| `market-data` | Keep only the concept of normalized read-only market data adapters. Do not migrate exchange-specific code directly. |
| `strategy` | No code should be migrated. At most preserve the idea that strategies consume normalized domain data. |
| `execution-sim` | Reuse only abstract order/request/result semantics as concepts, not real order placement logic. |
| `risk` | Reuse common error categories and validation ideas, especially around insufficient size and invalid exchange handling. |
| `backtest` | No direct module exists here; use the normalized types only as inspiration for historical/backtest interfaces. |
| `storage` | No reusable storage layer is present. Create fresh schemas and persistence design. |
| `agent-support` | Preserve factory/adapter registration ideas and documentation patterns for safe component assembly. |
| `docs` | Preserve warnings, safety disclaimers, normalization notes, and migration cautions. |

## 12. One smallest safe next step
Write a short design note in the future Rust repo that defines a **read-only normalized market-data domain contract** derived from this project's good ideas only: `PairId`, `Ticker`, `OrderBook`, and shared error categories. Exclude credentials, live order placement, exchange clients, and any trading workflow.
