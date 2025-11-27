---
title: Viewing Keys
description: Selective disclosure for compliance
---

# Viewing Keys

Viewing keys enable **selective disclosure** of shielded transaction details to authorized parties without compromising overall privacy.

## Use Cases

| Use Case | Description |
|----------|-------------|
| Tax Compliance | Share transaction history with tax authority |
| Audit | Allow auditor to verify specific transactions |
| Institutional | Meet regulatory requirements |
| Dispute Resolution | Prove transaction occurred to third party |
| Inheritance | Grant read access to estate executor |

## Privacy Model

```
Full Privacy (Shielded):     Only user sees transactions
Selective Disclosure:        User + authorized viewers see specific txs
Compliant Mode:             User + designated auditor see all txs
```

## Key Hierarchy

```
User Seed
    │
    └── Master Viewing Key (MVK)
            │
            ├── Full Viewing Key (FVK)
            │       └── Sees ALL transactions
            │
            ├── Auditor Key (AK)
            │       └── Sees ALL txs, time-limited
            │
            └── Transaction Viewing Keys (TVK)
                    └── Sees SINGLE transaction
```

### Master Viewing Key (MVK)

Root key derived from user's wallet seed:

```typescript
const mvk = generateViewingKey('/m/44/501/0')
```

### Full Viewing Key (FVK)

Grants access to ALL transactions:

```typescript
const fvk = deriveViewingKey(mvk, '/full')
```

**Warning**: FVK exposure reveals entire transaction history.

### Auditor Key (AK)

Time-limited viewing key for compliance:

```typescript
const auditKey = deriveViewingKey(mvk, '/audit/2024')
```

### Transaction Viewing Key (TVK)

Per-transaction key for selective disclosure:

```typescript
const tvk = deriveViewingKey(mvk, `/tx/${intentHash}`)
```

Reveals ONLY the specific transaction.

## Encryption Scheme

### Algorithm

| Component | Algorithm |
|-----------|-----------|
| Key derivation | HKDF-SHA256 |
| Symmetric encryption | XChaCha20-Poly1305 |
| Nonce | 24 bytes random |

### Encrypting Transaction Data

```typescript
import { encryptForViewing, decryptWithViewing } from '@sip-protocol/sdk'

const txData = {
  sender: '0xabc...',
  recipient: '0xdef...',
  amount: 1000n,
  asset: 'ETH'
}

// Encrypt for viewing key holder
const encrypted = encryptForViewing(txData, viewingKey)

// Decrypt with key
const decrypted = decryptWithViewing(encrypted, viewingKey)
```

### Multi-Key Encryption

Encrypt for multiple authorized viewers:

```typescript
const encrypted = encryptForViewing(txData, [
  auditorKey,
  complianceKey,
  taxKey
])
```

Each key holder can decrypt independently.

## Viewing Proof

A ViewingProof demonstrates that decrypted data is authentic without revealing the viewing key.

```typescript
interface ViewingProof {
  intentHash: string
  revealedData: TransactionData
  commitmentProof: {
    inputAmountProof: ZKProof
    outputAmountProof: ZKProof
  }
  viewingKeyHash: string
  proofTimestamp: number
}
```

Verifier checks:
1. ZK proof is valid
2. Revealed amounts match on-chain commitments
3. Viewing key hash matches authorized viewer

## Access Control

### Key Registration

```typescript
interface ViewingKeyGrant {
  grantId: string
  recipientId: string
  keyType: 'full' | 'auditor' | 'transaction'
  scope: {
    startTime?: number
    endTime?: number
    intentHashes?: string[]
  }
  createdAt: number
  revokedAt?: number
}
```

### Revocation

Keys can be revoked but previously-viewed data cannot be "un-revealed":

```typescript
await revokeViewingKey(grantId)
```

**Important**: Revocation prevents future access. Past disclosures are permanent.

## Security Properties

| Property | Guarantee |
|----------|-----------|
| Confidentiality | Only key holders can decrypt |
| Integrity | AEAD prevents tampering |
| Authenticity | ViewingProof proves data is real |
| Selective Disclosure | TVK reveals only one transaction |
| Forward Secrecy | Key compromise doesn't reveal past txs (with TVK) |

### What Viewing Keys CAN Do

- Decrypt transaction details
- Prove transaction authenticity
- Verify amounts match commitments

### What Viewing Keys CANNOT Do

- Spend funds
- Create new transactions
- Modify transaction history
- Reveal other users' transactions

## Integration with Privacy Levels

| Level | Viewing Key Behavior |
|-------|---------------------|
| TRANSPARENT | No encryption, public |
| SHIELDED | Encrypted, no viewing key shared |
| COMPLIANT | Encrypted, auditor key pre-shared |

## Workflow: Compliant Mode

```
1. User sets privacy_level = COMPLIANT
2. User pre-shares auditor_key with designated auditor
3. For each transaction:
   a. Encrypt tx_data with auditor_key
   b. Store encrypted blob with intent
   c. Auditor can decrypt at any time
4. If audit requested:
   a. Auditor decrypts all transactions
   b. Auditor generates ViewingProofs for report
```

## Workflow: Selective Disclosure

```
1. User has shielded transaction
2. Dispute arises, need to prove transaction
3. User derives TVK for specific intent
4. User shares TVK with arbiter
5. Arbiter decrypts ONLY that transaction
6. Arbiter generates ViewingProof
7. Dispute resolved with cryptographic proof
```

## Code Example

```typescript
import {
  generateViewingKey,
  deriveViewingKey,
  encryptForViewing,
  decryptWithViewing
} from '@sip-protocol/sdk'

// Generate master key from path
const masterKey = generateViewingKey('/m/44/501/0')

// Derive purpose-specific keys
const auditKey = deriveViewingKey(masterKey, '/audit/2024')
const taxKey = deriveViewingKey(masterKey, '/tax/quarterly')

// Encrypt transaction for auditor
const txData = {
  intentHash: '0x123...',
  inputAmount: 1000n,
  outputAmount: 950n,
  sender: '0xabc...',
  recipient: '0xdef...'
}

const encrypted = encryptForViewing(txData, auditKey)

// Auditor decrypts
const revealed = decryptWithViewing(encrypted, auditKey)
console.log(revealed.inputAmount) // 1000n
```

## Trust Model

| Key Type | Trust Required | Risk |
|----------|---------------|------|
| **FVK** | High | Full history exposure |
| **AK** | Medium | Bounded by time |
| **TVK** | Low | Single transaction |

## Comparison with Zcash

| Feature | Zcash | SIP |
|---------|-------|-----|
| Viewing key concept | Yes | Yes |
| Time-limited keys | No | Yes |
| ViewingProof | No | Yes |
| Multi-key encryption | No | Yes |
| Intent integration | No | Yes |
