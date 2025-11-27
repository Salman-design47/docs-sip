---
title: Architecture
description: System architecture and design of SIP Protocol
---

# Architecture

SIP Protocol operates as an application layer between user applications and the NEAR Intents settlement system.

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      APPLICATION LAYER                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   DApps     │  │   Wallets   │  │    DAOs     │              │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
│         └────────────────┼────────────────┘                      │
│                          ▼                                       │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    @sip-protocol/sdk                       │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────────┐  │  │
│  │  │ Intent  │ │ Stealth │ │ Privacy │ │ Wallet Adapters │  │  │
│  │  │ Builder │ │ Address │ │ Manager │ │ (ETH/SOL/NEAR)  │  │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                          │                                       │
├──────────────────────────┼───────────────────────────────────────┤
│                   PRIVACY LAYER (SIP)                            │
│  ┌───────────────────────┴───────────────────────────────────┐  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │  │
│  │  │   Pedersen   │  │   Stealth    │  │   Viewing Keys   │ │  │
│  │  │ Commitments  │  │  Addresses   │  │  (Compliance)    │ │  │
│  │  └──────────────┘  └──────────────┘  └──────────────────┘ │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │  │
│  │  │   Funding    │  │   Validity   │  │   Fulfillment    │ │  │
│  │  │    Proof     │  │    Proof     │  │     Proof        │ │  │
│  │  └──────────────┘  └──────────────┘  └──────────────────┘ │  │
│  └───────────────────────────────────────────────────────────┘  │
│                          │                                       │
├──────────────────────────┼───────────────────────────────────────┤
│                   SETTLEMENT LAYER                               │
│  ┌───────────────────────┴───────────────────────────────────┐  │
│  │              NEAR Intents + Chain Signatures               │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │  │
│  │  │  1Click API  │  │   Solvers    │  │ Chain Signatures │ │  │
│  │  └──────────────┘  └──────────────┘  └──────────────────┘ │  │
│  └───────────────────────────────────────────────────────────┘  │
│                          │                                       │
├──────────────────────────┼───────────────────────────────────────┤
│                    BLOCKCHAIN LAYER                              │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐    │
│  │  NEAR  │  │  ETH   │  │ Solana │  │ Zcash  │  │Bitcoin │    │
│  └────────┘  └────────┘  └────────┘  └────────┘  └────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## SDK Components

### SIP Client

The main entry point that orchestrates all operations:

```typescript
const sip = new SIP({
  network: 'testnet',
  proofProvider: new MockProofProvider(),
})
```

### IntentBuilder

Fluent API for constructing shielded intents:

```typescript
const intent = await sip
  .intent()
  .input('solana', 'SOL', amount)
  .output('ethereum', 'ETH')
  .privacy(PrivacyLevel.SHIELDED)
  .build()
```

### Wallet Adapters

Chain-specific wallet integrations:

| Adapter | Chain | Provider |
|---------|-------|----------|
| `EthereumWalletAdapter` | Ethereum | MetaMask, WalletConnect |
| `SolanaWalletAdapter` | Solana | Phantom, Solflare |
| `MockWalletAdapter` | Testing | Mock provider |

### Proof Providers

ZK proof generation interfaces:

| Provider | Status | Use Case |
|----------|--------|----------|
| `MockProofProvider` | Available | Testing, development |
| `NoirProofProvider` | Planned | Production |

## Data Flow

### Transparent Flow

```
User Intent → Standard intent → Solver → Settlement
```

No privacy features. All data visible on-chain.

### Shielded Flow

```
User Intent
    ↓
Generate stealth address (recipient privacy)
    ↓
Create Pedersen commitments (amount privacy)
    ↓
Generate ZK proofs (funding + validity)
    ↓
Submit shielded intent
    ↓
Solver fulfills (only sees commitments)
    ↓
Fulfillment proof generated
    ↓
Settlement to stealth address
```

### Compliant Flow

Same as shielded, plus:
- Encrypt transaction metadata with viewing key
- Auditor can decrypt specific transactions
- ViewingProof for audit reports

## Cryptographic Components

### Pedersen Commitments

```
C = value·G + blinding·H
```

- **Perfectly hiding**: Cannot extract value from C
- **Computationally binding**: Cannot change value after commit
- **Homomorphic**: C₁ + C₂ commits to sum of values

### Stealth Addresses

Based on EIP-5564:

1. Recipient publishes meta-address (P, Q)
2. Sender generates ephemeral key r
3. Sender computes stealth address A = Q + H(r·P)·G
4. Recipient scans for R values, derives private key

### Viewing Keys

Hierarchical key derivation:

```
Master Viewing Key
    ├── Full Viewing Key (all transactions)
    ├── Auditor Key (time-limited)
    └── Transaction Key (single transaction)
```

Encryption: XChaCha20-Poly1305 with HKDF key derivation.

## Network Adapters

### NEAR Intents Adapter

Connects to NEAR's 1Click API for cross-chain settlement:

```typescript
const adapter = new NEARIntentsAdapter({
  network: 'testnet',
  endpoint: 'https://1click.chaindefuser.com',
})
```

### Zcash Shielded Service

Handles Zcash-specific shielded transactions:

```typescript
const zcash = new ZcashShieldedService({
  rpcUrl: 'http://localhost:8232',
})
```

## Key Files

| File | Purpose |
|------|---------|
| `packages/sdk/src/sip.ts` | Main SIP client |
| `packages/sdk/src/intent.ts` | IntentBuilder |
| `packages/sdk/src/stealth.ts` | Stealth addresses |
| `packages/sdk/src/crypto.ts` | Pedersen commitments |
| `packages/sdk/src/privacy.ts` | Viewing keys |
| `packages/sdk/src/proofs/` | Proof providers |
| `packages/sdk/src/adapters/` | Network/wallet adapters |

## Performance

| Operation | Time (avg) |
|-----------|------------|
| Generate meta-address | 0.9ms |
| Derive stealth address | 5.4ms |
| Create commitment | 7.2ms |
| Verify commitment | 6.6ms |
| Full shielded intent | ~25ms |

Measured on Apple M1, Node.js 20.

## Tech Stack

- **Language**: TypeScript (strict)
- **Monorepo**: pnpm + Turborepo
- **Crypto**: @noble/curves, @noble/hashes, @noble/ciphers
- **Testing**: Vitest
- **CI/CD**: GitHub Actions
