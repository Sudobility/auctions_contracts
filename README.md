# @sudobility/auctions_contracts

Multi-chain auction system supporting Solana (native Rust `solana-program`) and EVM (Solidity/Hardhat). Implements Traditional, Dutch, and Penny auction types. Ships a TypeScript SDK that is the primary consumer-facing artifact (published to npm).

## Prerequisites

- **Node.js** >= 20 and **Bun** (package manager; `bun install`)
- **Rust / cargo** + Solana toolchain — required only to build/test the Solana program (`build:solana`, `test:solana`); Anchor CLI 0.28.0 is needed only for `compile:solana` and `deploy:solana:*`
- Installing `@sudobility/*` dependencies requires GitHub Packages auth (`.npmrc` maps the `@sudobility` scope to `npm.pkg.github.com`)

## Installation

```bash
bun install
```

## Usage

The package provides a TypeScript SDK with four export points:

```typescript
import { OnchainAuctionClient } from '@sudobility/auctions_contracts';          // Unified client
import { EvmAuctionClient } from '@sudobility/auctions_contracts/evm';          // EVM-specific (viem)
import { SolanaAuctionClient } from '@sudobility/auctions_contracts/solana';    // Solana-specific (web3.js)
import { ... } from '@sudobility/auctions_contracts/react-native';              // React Native entry (see note below)
```

> **Note:** The `react-native` export is declared in `package.json` but `src/react-native/` does not exist yet, so `dist/react-native/src/react-native/index.js` is not emitted. Use the default/`/evm`/`/solana` entries until the React Native entry is implemented.

### Auction Types

- **Traditional**: English-style ascending bids with reserve price and minimum increment
- **Dutch**: Descending price from start to minimum; first buyer wins immediately
- **Penny**: Fixed bid increment (pay-to-bid); each bid resets a 5-minute timer

### Supported Networks

- **EVM**: Ethereum, Sepolia, Polygon, Optimism, Base, Arbitrum
- **Solana**: Localnet, Devnet, Mainnet-beta

## Development

```bash
bun run build              # Build everything (EVM + Solana + Unified + React Native)
bun run build:ci           # Build unified + react-native only (no EVM/Solana compile)
bun run compile:evm        # Compile Solidity contracts only
bun run test:evm           # Run EVM tests (Hardhat + viem); aliases: test:unit, test:run
bun run test:solana        # Run Solana tests (cargo test)
bun run test:ci            # Build unified then run unified tests directly
bun run lint               # ESLint check
bun run lint:fix           # ESLint with auto-fix
bun run typecheck          # TypeScript compilation check
bun run format             # Prettier formatting
bun run clean              # Hardhat clean
```

### Deployment

```bash
bun run deploy:evm:local       # Deploy to Hardhat localhost
bun run deploy:evm:sepolia     # Deploy to Sepolia
bun run deploy:solana:devnet   # Deploy to Solana devnet
```

## Architecture

### EVM (Solidity 0.8.24)

- UUPS Upgradeable Proxy pattern
- Storage gap (`__gap`) for future slot reservation
- CEI pattern, SafeERC20, reentrancy guards

### Solana (native Rust, `solana-program` 1.18)

- Native program (manual `entrypoint!`/`declare_id!` + processor dispatch), **not** the Anchor framework — Anchor CLI is used only for deploy/`compile:solana`
- PDA-based accounting with deterministic address derivation
- Borsh 0.10 serialization, saturating arithmetic

### Unified Client

The `OnchainAuctionClient` uses dynamic `import()` to lazy-load platform-specific clients (EVM or Solana) on first use, keeping bundle size minimal.

## Key Constants

| Constant              | Value                   |
| --------------------- | ----------------------- |
| Fee Rate              | 0.5% (50 basis points)  |
| Max Fee Rate          | 10% (1000 basis points) |
| Acceptance Period     | 24 hours                |
| Penny Timer           | 5 minutes               |
| Max Items Per Auction | 255                     |

## License

BUSL-1.1
