---
title: Stealth Addresses
description: One-time addresses for recipient privacy
---

# Stealth Addresses

Stealth addresses provide **recipient privacy** by generating unique, one-time addresses for each transaction.

## The Problem

In current cross-chain swaps:

```
User has: shielded ZEC in z-address
User swaps: ZEC → SOL via intent
Refund goes to: t1ABC... (transparent, reused)

Chain analysis: "t1ABC received refunds 50 times"
               → Links to shielded activity
```

## The Solution

With SIP stealth addresses:

```
Each swap: generates fresh stealth address
Refund goes to: unique stealth_1, stealth_2, stealth_3...

Chain analysis: "No pattern - each address used once"
```

## How It Works

### Key Types

| Key | Symbol | Description |
|-----|--------|-------------|
| Spending private key | p | Controls spending |
| Spending public key | P = p·G | In meta-address |
| Viewing private key | q | Scans for incoming txs |
| Viewing public key | Q = q·G | In meta-address |
| Ephemeral key | r, R | Per-transaction |
| Stealth key | a, A | One-time address |

### Step 1: Recipient Generates Meta-Address

```typescript
const { metaAddress, spendingPrivateKey, viewingPrivateKey } =
  generateStealthMetaAddress('ethereum')

// Share meta-address publicly
console.log(metaAddress)
// sip:ethereum:0x02abc...def:0x03xyz...789
```

### Step 2: Sender Generates Stealth Address

```typescript
const { stealthAddress, ephemeralPublicKey } =
  generateStealthAddress(recipientMetaAddress)

// Send funds to stealthAddress.address
// Publish ephemeralPublicKey alongside transaction
```

### Step 3: Recipient Scans and Claims

```typescript
// Recipient scans for their transactions
const isMine = checkStealthAddress(
  stealthAddress,
  spendingPrivateKey,
  viewingPrivateKey
)

if (isMine) {
  // Derive private key to claim funds
  const privateKey = deriveStealthPrivateKey(
    stealthAddress,
    ephemeralPublicKey,
    spendingPrivateKey,
    viewingPrivateKey
  )
}
```

## Cryptographic Protocol

### Key Generation

```
KeyGen():
  1. p ← random_scalar()     // Spending private
  2. P ← p · G               // Spending public
  3. q ← random_scalar()     // Viewing private
  4. Q ← q · G               // Viewing public
  Return: ((P, Q), p, q)
```

### Stealth Address Generation

```
GenerateStealth(P, Q):
  1. r ← random_scalar()     // Ephemeral private
  2. R ← r · G               // Ephemeral public
  3. S ← r · P               // Shared secret (ECDH)
  4. h ← SHA256(S)           // Hash shared secret
  5. view_tag ← h[0]         // First byte (optimization)
  6. A ← Q + h · G           // Stealth address
  Return: (A, R, view_tag)
```

### Scanning

```
ScanForStealth(R, A, view_tag, p, q):
  1. S' ← p · R              // Recompute shared secret
  2. h' ← SHA256(S')

  // Quick reject using view tag
  3. if h'[0] ≠ view_tag: return NOT_MINE

  // Full verification
  4. A' ← Q + h' · G
  5. if A' ≠ A: return NOT_MINE
  Return: MINE
```

### Private Key Recovery

```
DerivePrivateKey(R, p, q):
  1. S ← p · R
  2. h ← SHA256(S)
  3. a ← q + h (mod n)       // Stealth private key
  Return: a
```

## Encoding Format

### Meta-Address

```
sip:<chain>:<spending_key>:<viewing_key>

Example:
sip:ethereum:0x02abc...def:0x03123...456
```

### Supported Chains

| Chain ID | Description |
|----------|-------------|
| `ethereum` | Ethereum mainnet |
| `solana` | Solana mainnet |
| `zcash` | Zcash (t-addresses) |
| `near` | NEAR Protocol |
| `polygon` | Polygon PoS |

## Security Properties

| Property | Guarantee |
|----------|-----------|
| **Unlinkability** | Different addresses cannot be linked |
| **Recipient Privacy** | Observer cannot determine recipient |
| **Sender Deniability** | Anyone could have generated the address |
| **Forward Secrecy** | Viewing key compromise doesn't reveal past |

### What is Protected

| Information | Protected? |
|-------------|------------|
| Recipient identity | Yes |
| Link between txs | Yes |
| Recipient's balance | Yes |
| Viewing key holder | Yes |

### What is NOT Protected

| Information | Protected? | Notes |
|-------------|------------|-------|
| Transaction amount | No | Use commitments |
| Sender identity | No | Stealth is for recipient |
| Transaction timing | No | Block timestamp visible |

## View Tag Optimization

The 8-bit view tag reduces scanning work by 256x:

- Only 1/256 addresses need full verification
- Leaks 8 bits of information (acceptable tradeoff)
- Addresses sharing view tag are still unlinkable

## Chain-Specific Derivation

Different chains derive addresses differently from stealth public key A:

| Chain | Derivation |
|-------|------------|
| Ethereum | `keccak256(A)[12:32]` |
| Zcash (t-addr) | `hash160(A)` + version |
| Bitcoin | `hash160(A)` + version |

## Code Example

```typescript
import {
  generateStealthMetaAddress,
  generateStealthAddress,
  checkStealthAddress,
  deriveStealthPrivateKey,
  encodeStealthMetaAddress,
  decodeStealthMetaAddress
} from '@sip-protocol/sdk'

// Recipient setup (one-time)
const meta = generateStealthMetaAddress('ethereum')
const encoded = encodeStealthMetaAddress(meta.metaAddress)
// Share `encoded` publicly

// Sender creates payment
const decoded = decodeStealthMetaAddress(encoded)
const { stealthAddress, ephemeralPublicKey } =
  generateStealthAddress(decoded)

// Recipient scans
const isMine = checkStealthAddress(
  stealthAddress,
  meta.spendingPrivateKey,
  meta.viewingPrivateKey
)

// Recipient claims
if (isMine) {
  const pk = deriveStealthPrivateKey(
    stealthAddress,
    ephemeralPublicKey,
    meta.spendingPrivateKey,
    meta.viewingPrivateKey
  )
  // Use pk to spend from stealthAddress
}
```

## EIP-5564 Compatibility

SIP stealth addresses align with EIP-5564:

| Aspect | EIP-5564 | SIP |
|--------|----------|-----|
| Curve | secp256k1 | secp256k1 |
| Key structure | (P, Q) | (P, Q) |
| Shared secret | ECDH | ECDH |
| View tag | 1 byte | 1 byte |

**Extension**: SIP adds multi-chain support via `sip:` URI scheme.
