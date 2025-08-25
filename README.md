# HyperSwap Protocol - Advanced DeFi Liquidity Engine

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Clarity](https://img.shields.io/badge/Clarity-3.0-blue.svg)](https://clarity-lang.org/)
[![Tests](https://img.shields.io/badge/Tests-Vitest-green.svg)](https://vitest.dev/)
[![Stacks](https://img.shields.io/badge/Stacks-Blockchain-purple.svg)](https://www.stacks.co/)

Revolutionary permissionless liquidity protocol powering seamless token swaps and yield generation through intelligent automated market making technology on the Stacks blockchain.

## 🌟 Key Features

### Core Capabilities

- **Zero-Permission Pool Deployment** - Launch any token pair instantly without approval
- **Intelligent Price Discovery** - Advanced Constant Product Market Maker (CPMM) algorithms minimize slippage
- **Yield-Optimized Liquidity** - Proportional rewards with compounding benefits for liquidity providers
- **Dynamic Fee Architecture** - Self-adjusting rates for optimal market conditions (0.3% default)
- **Battle-Tested Security** - Multi-layer protection with emergency controls and ownership validation
- **Gas-Optimized Operations** - Maximum efficiency for every transaction with 6-decimal precision
- **Governance-Ready Framework** - Community-driven protocol evolution capabilities

### Technical Highlights

- **Precision**: 6-decimal precision (`1,000,000`) for accurate calculations
- **Security**: Comprehensive error handling with 7 distinct error codes
- **Flexibility**: Support for any SIP-010 compliant fungible tokens
- **Emergency Controls**: Pool pause/resume functionality for risk management
- **LP Token System**: Proportional ownership tracking with automatic minting/burning

## 🏗️ Architecture

### Smart Contract Structure

```
contracts/
└── hyper-swap.clar          # Main protocol contract
```

### Core Components

#### 1. Fungible Token Interface (SIP-010)

```clarity
(define-trait ft-trait (
  (transfer (uint principal principal) (response bool uint))
  (get-balance (principal) (response uint uint))
  (get-total-supply () (response uint uint))
  (get-decimals () (response uint uint))
  (get-name () (response (string-ascii 32) uint))
  (get-symbol () (response (string-ascii 32) uint))
))
```

#### 2. Pool Data Structure

```clarity
{
  token-x: principal,        # First token contract
  token-y: principal,        # Second token contract
  reserve-x: uint,           # Reserve amount of token X
  reserve-y: uint,           # Reserve amount of token Y
  total-shares: uint,        # Total LP tokens in circulation
  active: bool,              # Pool operational status
}
```

#### 3. Liquidity Provider Tracking

```clarity
{
  pool-id: uint,            # Pool identifier
  provider: principal,      # LP address
}
→ { shares: uint }          # LP token balance
```

## 🚀 Quick Start

### Prerequisites

- [Clarinet CLI](https://github.com/hirosystems/clarinet) installed
- [Node.js](https://nodejs.org/) v16+ for testing
- Basic understanding of Clarity smart contracts

### Installation

1. **Clone the repository:**

```bash
git clone https://github.com/carlie-brent/hyper-swap.git
cd hyper-swap
```

2. **Install dependencies:**

```bash
npm install
```

3. **Check contract validity:**

```bash
clarinet check
```

4. **Run tests:**

```bash
npm test
```

### Development Setup

1. **Format contracts:**

```bash
clarinet fmt --in-place
```

2. **Run tests with coverage:**

```bash
npm run test:report
```

3. **Watch mode for development:**

```bash
npm run test:watch
```

## 📚 API Reference

### Public Functions

#### Pool Management

##### `create-pool`

Creates a new liquidity pool for a token pair.

```clarity
(create-pool (token-x <ft-trait>) (token-y <ft-trait>)) → (response uint uint)
```

**Parameters:**

- `token-x`: First token contract implementing ft-trait
- `token-y`: Second token contract implementing ft-trait

**Returns:** Pool ID on success

**Requirements:**

- Caller must be contract owner
- Tokens must be different contracts

---

##### `add-liquidity`

Adds liquidity to an existing pool and receives LP tokens.

```clarity
(add-liquidity 
  (pool-id uint) 
  (token-x <ft-trait>) 
  (token-y <ft-trait>) 
  (amount-x uint) 
  (amount-y uint) 
  (min-shares uint)
) → (response uint uint)
```

**Parameters:**

- `pool-id`: Target pool identifier
- `token-x`, `token-y`: Token contracts (must match pool)
- `amount-x`, `amount-y`: Token amounts to deposit
- `min-shares`: Minimum LP tokens to receive (slippage protection)

**Returns:** Number of LP tokens minted

---

##### `remove-liquidity`

Removes liquidity and redeems underlying tokens.

```clarity
(remove-liquidity 
  (pool-id uint) 
  (token-x <ft-trait>) 
  (token-y <ft-trait>) 
  (shares uint) 
  (min-amount-x uint) 
  (min-amount-y uint)
) → (response {amount-x: uint, amount-y: uint} uint)
```

**Parameters:**

- `pool-id`: Target pool identifier
- `token-x`, `token-y`: Token contracts (must match pool)
- `shares`: LP tokens to burn
- `min-amount-x`, `min-amount-y`: Minimum tokens to receive (slippage protection)

**Returns:** Amounts of tokens received

#### Trading

##### `swap-exact-tokens`

Executes token swap with slippage protection.

```clarity
(swap-exact-tokens 
  (pool-id uint) 
  (token-in <ft-trait>) 
  (token-out <ft-trait>) 
  (amount-in uint) 
  (min-amount-out uint) 
  (x-to-y bool)
) → (response uint uint)
```

**Parameters:**

- `pool-id`: Target pool identifier
- `token-in`, `token-out`: Input and output token contracts
- `amount-in`: Exact input amount
- `min-amount-out`: Minimum output amount (slippage protection)
- `x-to-y`: Direction flag (true = X→Y, false = Y→X)

**Returns:** Actual output amount

### Read-Only Functions

##### `get-pool-info`

Retrieves complete pool information.

```clarity
(get-pool-info (pool-id uint)) → (optional pool-data)
```

##### `get-provider-shares`

Gets LP token balance for specific provider.

```clarity
(get-provider-shares (pool-id uint) (provider principal)) → (optional {shares: uint})
```

##### `get-exchange-rate`

Calculates current exchange rate for pool.

```clarity
(get-exchange-rate (pool-id uint)) → (response uint uint)
```

### Administrative Functions

##### `set-protocol-fee`

Updates protocol fee rate (owner only).

```clarity
(set-protocol-fee (new-fee uint)) → (response bool uint)
```

##### `pause-pool` / `resume-pool`

Emergency pool control functions.

```clarity
(pause-pool (pool-id uint)) → (response bool uint)
(resume-pool (pool-id uint)) → (response bool uint)
```

## 🧮 Mathematical Model

### Constant Product Formula

HyperSwap implements the proven constant product automated market maker model:

```
x * y = k (constant)
```

Where:

- `x` = reserve of token X
- `y` = reserve of token Y  
- `k` = constant product

### Output Calculation

For a swap of input amount `Δx`, the output amount `Δy` is:

```
Δy = (y * Δx * (1 - fee)) / (x + Δx * (1 - fee))
```

Where:

- `fee` = protocol fee rate (default 0.3%)
- Precision: 1,000,000 (6 decimals)

### LP Token Valuation

For the first liquidity provision:

```
shares = √(amount_x * amount_y)
```

For subsequent provisions:

```
shares = min(
  (amount_x * total_shares) / reserve_x,
  (amount_y * total_shares) / reserve_y
)
```

## 🛡️ Security Features

### Access Control

- **Owner-only functions**: Pool creation, fee updates, emergency controls
- **Validation checks**: Token contract verification, amount validation
- **State verification**: Pool existence, activation status

### Error Handling

```clarity
ERR-NOT-AUTHORIZED      (u100)  # Unauthorized access
ERR-INVALID-AMOUNT      (u101)  # Invalid amount parameter
ERR-INSUFFICIENT-BALANCE (u102)  # Insufficient balance
ERR-POOL-NOT-FOUND      (u103)  # Pool doesn't exist
ERR-INVALID-POOL        (u104)  # Invalid pool configuration
ERR-SLIPPAGE-TOO-HIGH   (u105)  # Slippage protection triggered
ERR-ZERO-LIQUIDITY      (u106)  # Zero liquidity error
```

### Emergency Controls

- **Pool Pausing**: Immediate halt of all pool operations
- **Owner Override**: Administrative control for risk management
- **Slippage Protection**: Minimum output guarantees

## 🧪 Testing

### Test Structure

```
tests/
└── hyper-swap.test.ts       # Comprehensive test suite
```

### Running Tests

```bash
# Run all tests
npm test

# Run with coverage report
npm run test:report

# Watch mode for development
npm run test:watch

# Check contract validity
clarinet check
```

### Test Categories

- **Pool Creation**: Validation and initialization
- **Liquidity Management**: Add/remove operations
- **Trading Logic**: Swap calculations and execution
- **Security**: Access control and error handling
- **Edge Cases**: Boundary conditions and error states

## 🔧 Configuration

### Clarinet Configuration

```toml
[project]
name = 'hyper-swap'
clarity_version = 3
epoch = 'latest'

[contracts.hyper-swap]
path = 'contracts/hyper-swap.clar'
```

### Network Settings

- **Devnet**: Local development network
- **Testnet**: Stacks testnet deployment
- **Mainnet**: Production deployment

## 📈 Performance Metrics

### Gas Optimization

- **Efficient calculations**: Minimal computational overhead
- **Optimized storage**: Compact data structures
- **Batch operations**: Grouped state updates

### Precision Standards

- **6-decimal precision**: Balances accuracy with efficiency
- **Integer arithmetic**: No floating-point vulnerabilities
- **Overflow protection**: Safe mathematical operations

## 🤝 Contributing

We welcome contributions to the HyperSwap Protocol! Please follow these guidelines:

### Development Process

1. Fork the repository
2. Create a feature branch
3. Implement changes with tests
4. Ensure all tests pass
5. Submit a pull request

### Code Standards

- Follow Clarity best practices
- Include comprehensive tests
- Document all public functions
- Maintain security standards

### Testing Requirements

- Unit tests for all functions
- Integration tests for workflows
- Security tests for access control
- Edge case coverage

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🔗 Links

- **Documentation**: [Clarity Language Reference](https://docs.stacks.co/clarity/)
- **Stacks Blockchain**: [Official Website](https://www.stacks.co/)
- **Clarinet CLI**: [GitHub Repository](https://github.com/hirosystems/clarinet)
- **SIP-010**: [Fungible Token Standard](https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md)
