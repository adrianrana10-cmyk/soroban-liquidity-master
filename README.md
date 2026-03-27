# SoroPool

> **Constant-product AMM on Soroban — two-token liquidity pool with a 0.3% swap fee.**

A Soroban smart contract implementing a minimal constant-product automated market maker. Liquidity providers deposit token pairs and receive shares representing their proportional claim. Traders swap against the pool at algorithmically derived prices with slippage protection built in.

---

## Hackathon Info

| Field | Details |
|---|---|
| **Track** | DeFi / Finance |
| **Chain** | Stellar (Soroban) |
| **Target users** | Liquidity providers, DeFi traders, protocol developers |
| **Status** | Hackathon MVP |

---

## Problem

Stellar's native DEX is order-book based — there is no on-chain AMM primitive for developers to build DeFi protocols on top of. Teams that want programmable liquidity, composable swaps, or LP share tokens have no reusable starting point in the Soroban ecosystem.

---

## Solution

**SoroPool** is a self-contained constant-product AMM deployed as a single Soroban contract. It accepts two SEP-0041 tokens, maintains on-chain reserves, and issues transferable liquidity shares. Any address can provide liquidity, swap tokens, or withdraw their position — all enforced on-chain with no admin key.

---

### Pool mechanics

The pool enforces the constant-product invariant across every swap:

```
reserve_a * reserve_b = k  (constant)
```

The 0.3% fee is applied by using only 99.7% of the input amount in the invariant calculation:

```
effective_input = input_amount * 997 / 1000
```

Liquidity shares are minted proportionally on deposit and burned on withdrawal, giving each LP a claim on the pool's reserves at all times.

---

## Contract API

### Entry points

| Function | Access | Description |
|---|---|---|
| `__constructor(token_a, token_b)` | Deploy-time | Initializes pool; panics if `token_a >= token_b` |
| `deposit(to, desired_a, min_a, desired_b, min_b)` | Any LP | Deposits tokens, mints proportional shares |
| `swap(to, buy_a, out, in_max)` | Any trader | Swaps tokens using constant-product invariant |
| `withdraw(to, share_amount, min_a, min_b)` | Share holder | Burns shares, returns proportional token amounts |
| `balance_shares(user)` | Public | Returns LP share balance for an address |
| `get_rsrvs()` | Public | Returns current `(reserve_a, reserve_b)` |

### Swap formula

```
// buying token_a (sell token_b)
sell_amount = (reserve_b * out * 1000) / ((reserve_a - out) * 997) + 1

// buying token_b (sell token_a)
sell_amount = (reserve_a * out * 1000) / ((reserve_b - out) * 997) + 1
```

The trade reverts if `sell_amount > in_max`.

### Token ordering

The constructor enforces `token_a < token_b` by address comparison. Deploying out of order will panic.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Core contract** | Rust (Soroban) — AMM logic, share minting, reserve tracking |
| **Token interface** | SEP-0041 compatible (any two Soroban tokens) |
| **Auth** | Soroban native authorization per depositor, swapper, withdrawer |
| **Frontend** | React + Stellar Wallets Kit — swap UI, LP dashboard, reserve display |

---

## Why Stellar / Soroban?

- **Atomic token transfers** — deposit, swap, and withdraw settle in a single transaction
- **SEP-0041 token standard** — any Soroban token pair works without modification
- **No admin key** — pool is fully permissionless after deployment
---

## Deployment

The SoroPool contract has been deployed on the Stellar testnet for demonstration purposes.

### Testnet Deployment
- **Contract ID**: `CCZZYGWWFQXYVBFLQNIFDUYOA7TTV2IMSOFEQIEF2V63HXQHUAXCBP35`
- **Network**: Stellar Testnet
- **Token A**: `CCF7V7MZ7VA3HOKSWP34VG26XZZ7I2D7Q4YY5CS6SLBWH3WM6MIERELD` (TESTB asset)
- **Token B**: `CC2S7GFRQYHNPITBRWYJ6AJKE76Y2RED6O3SA3UY673GOJYEYOH5YLW7` (TESTA asset)

### Contract Links
- **Stellar Expert**: https://stellar.expert/explorer/testnet/contract/CCZZYGWWFQXYVBFLQNIFDUYOA7TTV2IMSOFEQIEF2V63HXQHUAXCBP35
- **Stellar Lab**: https://lab.stellar.org/r/testnet/contract/CCZZYGWWFQXYVBFLQNIFDUYOA7TTV2IMSOFEQIEF2V63HXQHUAXCBP35
- **Deployment Transaction**: https://stellar.expert/explorer/testnet/tx/48c360a268233549f4ddca23c814eb48f88b9527c67fcc4044406a08d7d1318c

---

## Project Structure

```
soroban-liquidity-pool/
├── src/
│   ├── lib.rs       # Contract implementation
│   └── test.rs      # Unit tests (Soroban testutils)
├── Cargo.toml
└── README.md
```

---

## Develop

Run tests:

```bash
cargo test
```

Build (WASM):

```bash
cargo build --release --target wasm32-unknown-unknown
```

---

## Hackathon MVP Checklist

- [ ] `deposit`, `swap`, and `withdraw` entry points with slippage protection
- [ ] Constant-product invariant with 0.3% fee applied correctly
- [ ] LP share minting and burning proportional to reserves
- [ ] Token ordering enforced at deployment
- [ ] LP dashboard: deposit tokens, view share balance, withdraw position
- [ ] Swap UI: select direction, input amount, preview output, execute
- [ ] Live demo: two-token pool on Stellar Testnet with real reserve tracking

---

## Differentiators

| Feature | SoroPool | Order-book DEX |
|---|---|---|
| Continuous liquidity | Yes — always a price | No — requires matching orders |
| LP participation | Yes — any address | No |
| Programmatic price | Yes — x * y = k | No — order-driven |
| Slippage protection | Built-in `in_max` / `min` checks | Manual |
| Admin required | None post-deploy | Varies |
| Composable shares | Yes — transferable LP tokens | No |

---

## Roadmap (Post-Hackathon)

- **V2 — Configurable fee** — fee tier set at deployment, allowing pools with 0.05%, 0.3%, or 1% fee
- **V3 — Multi-hop routing** — chain multiple SoroPool instances for token pairs without a direct pool
- **V4 — Concentrated liquidity** — LPs specify price ranges, improving capital efficiency over full-range AMM

---

## Contributing

This repo is a hackathon starting point. PRs, issues, and ideas welcome.

---

## License

MIT
