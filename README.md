# SoroPool - Constant Product AMM on Soroban

[![Stellar](https://img.shields.io/badge/Stellar-Soroban-blue)](https://soroban.stellar.org)
[![Rust](https://img.shields.io/badge/Rust-1.89+-orange)](https://www.rust-lang.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

> **A minimal constant-product automated market maker (AMM) for Stellar's Soroban smart contract platform**

SoroPool implements a decentralized exchange liquidity pool with a 0.3% swap fee, enabling permissionless token trading on Stellar. Built as a single Soroban contract, it provides the foundational DeFi primitive missing from Stellar's native order-book DEX.

## 🚀 Features

- **Constant Product AMM**: Implements the `x * y = k` invariant
- **0.3% Swap Fee**: Industry-standard fee structure
- **Liquidity Shares**: Mint/burn transferable LP tokens
- **Slippage Protection**: Configurable minimum output amounts
- **Permissionless**: No admin keys or centralized control
- **SEP-0041 Compatible**: Works with any Soroban token standard

## 📊 How It Works

### Pool Mechanics
The pool maintains two token reserves (`reserve_a`, `reserve_b`) that must satisfy:
```
reserve_a × reserve_b = k  (constant)
```

### Swapping
When swapping tokens, the contract calculates the required input amount using:
```
// Buying token_a (selling token_b)
sell_amount = (reserve_b × out × 1000) / ((reserve_a - out) × 997) + 1

// Buying token_b (selling token_a)
sell_amount = (reserve_a × out × 1000) / ((reserve_b - out) × 997) + 1
```

The 0.3% fee is applied by using only 99.7% of the input in the calculation.

### Liquidity Provision
- Deposit equal values of both tokens (first deposit sets ratio)
- Receive LP shares proportional to your contribution
- Withdraw anytime by burning shares

## 🛠 Setup & Installation

### Prerequisites
- [Rust](https://rust-lang.org) 1.89+
- [Stellar CLI](https://soroban.stellar.org/docs/getting-started/setup) (Soroban tools)
- Funded Stellar testnet account

### 1. Install Stellar CLI
```bash
# Install via cargo
cargo install stellar-cli

# Or download from releases
# https://github.com/stellar/stellar-cli/releases
```

### 2. Setup Rust Target
```bash
rustup target add wasm32v1-none
```

### 3. Clone & Build
```bash
git clone https://github.com/adrianrana10-cmyk/soroban-liquidity-master.git
cd soroban-liquidity-master

# Build the contract
stellar contract build
```

### 4. Run Tests
```bash
cargo test
```

## 🚀 Deployment

### Testnet Deployment
The contract is already deployed on Stellar testnet:

- **Contract ID**: `CCZZYGWWFQXYVBFLQNIFDUYOA7TTV2IMSOFEQIEF2V63HXQHUAXCBP35`
- **Network**: Testnet
- **Token A**: `CCF7V7MZ7VA3HOKSWP34VG26XZZ7I2D7Q4YY5CS6SLBWH3WM6MIERELD` (TESTB)
- **Token B**: `CC2S7GFRQYHNPITBRWYJ6AJKE76Y2RED6O3SA3UY673GOJYEYOH5YLW7` (TESTA)

### Deploy Your Own Instance

#### 1. Configure Network
```bash
# Add testnet
stellar network add --global testnet \
  --rpc-url https://soroban-testnet.stellar.org \
  --network-passphrase "Test SDF Network ; September 2015"
```

#### 2. Create/Fund Account
```bash
# Generate new identity
stellar keys generate my-account

# Fund with testnet XLM
# Visit: https://faucet.stellar.org/
# Enter your address: stellar keys address my-account
```

#### 3. Deploy Token Contracts
```bash
# Deploy token A
stellar contract asset deploy \
  --asset MYTOKENA:$(stellar keys address my-account) \
  --source my-account \
  --network testnet

# Deploy token B
stellar contract asset deploy \
  --asset MYTOKENB:$(stellar keys address my-account) \
  --source my-account \
  --network testnet
```

#### 4. Deploy AMM Contract
```bash
# Note: Ensure token_a < token_b lexicographically
stellar contract deploy \
  --wasm target/wasm32v1-none/release/soroban_liquidity_pool_contract.wasm \
  --source my-account \
  --network testnet \
  -- --token_a <TOKEN_A_ADDRESS> --token_b <TOKEN_B_ADDRESS>
```

## 📖 Usage

### Contract Interface

| Function | Parameters | Description |
|----------|------------|-------------|
| `deposit` | `to, desired_a, min_a, desired_b, min_b` | Add liquidity, mint LP shares |
| `swap` | `to, buy_a, out, in_max` | Swap tokens (buy_a=true for token_a) |
| `withdraw` | `to, share_amount, min_a, min_b` | Remove liquidity, burn shares |
| `balance_shares` | `user` | Get LP share balance |
| `get_rsrvs` | - | Get current reserves |

### Example Interactions

#### Check Reserves
```bash
stellar contract invoke \
  --id CCZZYGWWFQXYVBFLQNIFDUYOA7TTV2IMSOFEQIEF2V63HXQHUAXCBP35 \
  --source my-account \
  --network testnet \
  -- get_rsrvs
```

#### Mint Tokens (for testing)
```bash
# Mint token A to your account
stellar contract invoke \
  --id <TOKEN_A_CONTRACT> \
  --source my-account \
  --network testnet \
  -- mint --to $(stellar keys address my-account) --amount 1000000000
```

#### Add Liquidity
```bash
stellar contract invoke \
  --id CCZZYGWWFQXYVBFLQNIFDUYOA7TTV2IMSOFEQIEF2V63HXQHUAXCBP35 \
  --source my-account \
  --network testnet \
  -- deposit \
  --to $(stellar keys address my-account) \
  --desired_a 1000000000 \
  --min_a 1000000000 \
  --desired_b 1000000000 \
  --min_b 1000000000
```

#### Swap Tokens
```bash
# Swap 100 token_b for token_a
stellar contract invoke \
  --id CCZZYGWWFQXYVBFLQNIFDUYOA7TTV2IMSOFEQIEF2V63HXQHUAXCBP35 \
  --source my-account \
  --network testnet \
  -- swap \
  --to $(stellar keys address my-account) \
  --buy_a true \
  --out 100000000 \
  --in_max 200000000
```

## 🔍 Contract Architecture

### Storage Keys
- `TokenA`: Address of first token
- `TokenB`: Address of second token
- `TotalShares`: Total LP shares minted
- `ReserveA`: Current reserve of token A
- `ReserveB`: Current reserve of token B
- `Shares(user)`: LP shares per user

### Security Features
- **Authorization**: All state-changing functions require user auth
- **Slippage Protection**: Minimum output amounts prevent sandwich attacks
- **Overflow Protection**: Uses checked arithmetic
- **Token Ordering**: Enforces `token_a < token_b` to prevent duplicates

## 🧪 Testing

Run the test suite:
```bash
cargo test
```

Tests cover:
- Basic deposit/withdraw functionality
- Swap calculations
- Edge cases (zero amounts, insufficient shares)
- Authorization requirements

## 🌐 Links

- **Live Contract**: https://stellar.expert/explorer/testnet/contract/CCZZYGWWFQXYVBFLQNIFDUYOA7TTV2IMSOFEQIEF2V63HXQHUAXCBP35
- **Stellar Lab**: https://lab.stellar.org/r/testnet/contract/CCZZYGWWFQXYVBFLQNIFDUYOA7TTV2IMSOFEQIEF2V63HXQHUAXCBP35
- **Soroban Docs**: https://soroban.stellar.org/docs
- **Stellar Developer**: https://developers.stellar.org

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ⚠️ Disclaimer

This is experimental software for educational and testing purposes. Use at your own risk. The contract has not been audited for production use.