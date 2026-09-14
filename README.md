<div align="center">
  <h1>🛡️ Soroban Cost Guard</h1>
  <p><strong>Empirical cost measurement and assertion tooling for Soroban smart contracts.</strong></p>

  [![Build Status](https://github.com/soroban-cost-guard/soroban-cost-guard/actions/workflows/budget.yml/badge.svg)](https://github.com/soroban-cost-guard/soroban-cost-guard/actions/workflows/budget.yml)
  [![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
  <p>
    <a href="https://tollcraft.gitbook.io/docs/cost-guard"><strong>Documentation</strong></a> ·
    <a href="https://soroban-cost-guard.github.io/soroban-cost-guard/dashboard.html"><strong>Dashboard</strong></a> ·
    <a href="https://asciinema.org/a/qqC0RysuCDBvfUXC"><strong>Demo</strong></a>
  </p>
</div>

---

## 📖 Overview

Soroban Cost Guard is empirical cost measurement and assertion tooling for Soroban smart contracts. Catches resource regressions in CI before they hit the network — using macro-level local assertions (Tier A) and real testnet simulation via `cargo budget-report` (Tier B). Built for Stellar Protocol 22.

### 🏗️ Two-Tier Architecture

| Tier | Crate | What it does |
| :--- | :---- | :----------- |
| **A — Local** | `crates/macros` | Rust attribute macros (`#[budget_cpu_lt(N)]`, `#[budget_mem_lt(N)]`) that fail `cargo test` the moment cost crosses your pinned limit. Fast, offline, CI-blocking. |
| **B — Network** | `cli` (`cargo-budget-report`) | Discovers all contracts in the workspace, compiles WASM, simulates on testnet, and reports CPU instructions + read/write bytes. Configurable via `budget.toml`. |

### 🧪 Test Fixture

The workspace ships an `amm-pool` constant-product AMM fixture under `contracts/fixtures/`. It exercises persistent storage, `require_auth()`, event emission, and constant-product math — the operations that dominate real Soroban costs. A synthetic `do_expensive_work` loop is kept as a comparison baseline.

---

## 🚀 Quick Start

### 1. Install

```bash
cargo install cargo-budget-report
```

### 2. Configure `budget.toml`

```bash
cargo budget-report --init
```

```toml
network = "testnet"
source = "alice"

[functions.do_expensive_work]
args = ["--n", "10000"]
cpu_limit = 5000000
read_limit = 5000
write_limit = 1000
```

### 3. Run

```bash
# Full workspace report
cargo budget-report

# Enforce limits — exits non-zero on any breach
cargo budget-report --check

# Machine-readable output
cargo budget-report --check --json
```

### 4. Add Macro Assertions

```rust
use budget_macros::budget_cpu_lt;

#[test]
#[budget_cpu_lt(env_file = "../../../tier-a-limits.env",
               env = "TIER_A__AMM_POOL_CONTRACT__SCENARIO__FULL_WORKFLOW__CPU")]
fn test_cpu_budget() {
    let env = Env::default();
    // ... your contract invocation ...
}
```

---

## ⚙️ Compatibility

| SDK Version    | Protocol | Status        |
| :------------- | :------- | :------------ |
| `< 22.0.0`     | `< 22`   | **Untested**  |
| `22.0.x`       | `22`     | **Supported** |
| `>= 23.0.0`    | `>= 23`  | **Untested**  |

See [Cargo.toml](Cargo.toml) for pinned manifest dependencies per crate.

---

## 📊 Dashboard

Every push to `main` records a real network measurement to `history.json` on `gh-pages`. The static dashboard at [`site/dashboard.html`](site/dashboard.html) plots per-function cost trend lines across commits — every point is a genuine testnet simulation, never mocked.

**Use it in your own repo:** copy the `record-history` job from [`.github/workflows/budget.yml`](.github/workflows/budget.yml) and the `site/` folder, then open:

```
https://your-org.github.io/your-repo/dashboard.html?limit=100
```

---

## 🤝 Community & Maintainers

- **Discord**: [Stellar Developer Discord](https://discord.gg/5aprtMSyR)
- **Telegram**: [@tollcraft](https://t.me/+Gflo5jZStw1jMjE0)

[![Contributors](https://contrib.rocks/image?repo=soroban-cost-guard/soroban-cost-guard)](https://github.com/soroban-cost-guard/soroban-cost-guard/graphs/contributors)

---

## 🛠️ Contributing

We welcome contributions. See [CONTRIBUTING.md](CONTRIBUTING.md) to get started and [SECURITY.md](SECURITY.md) for reporting vulnerabilities.

**Generate the CLI man page:**

```bash
cargo run -p cargo-budget-report --bin generate-manpage -- cargo-budget-report.1
```
