---
title: Known Limitations
description: Current limitations and planned mitigations
---

# Known Limitations

This document describes the current limitations of SIP Protocol and planned mitigations.

## Privacy Limitations

### Timing Correlation

**Issue**: Transactions submitted close in time may be correlated by observers.

**Impact**: Medium - Reduces anonymity set size

**Mitigation**:
- Delayed submission (not implemented)
- Transaction batching (future)
- Mixing services (external)

**Recommendation**: Don't submit multiple related transactions in rapid succession.

### Amount Inference from Output

**Issue**: When output amount is public (for quoting), the input amount may be inferable via market rates.

**Impact**: Low-Medium - Partial information leakage

**Mitigation**:
- Output range commitments (future)
- Decoy amounts (not implemented)

**Recommendation**: Use SHIELDED mode when exact amounts are sensitive.

### View Tag Information Leakage

**Issue**: The 8-bit view tag reveals some information about shared secret.

**Impact**: Low - Only 8 bits, insufficient for linkability

**Mitigation**: Accepted tradeoff for 256x scanning improvement.

**Recommendation**: None needed - acceptable privacy cost.

### Cross-Chain Timing Analysis

**Issue**: Source and destination chain transactions may be correlated by timing.

**Impact**: Medium - Especially for low-traffic chains

**Mitigation**:
- Random delays (not implemented)
- Batched settlement (future)

**Recommendation**: Avoid immediate swaps for high-value privacy needs.

## Implementation Limitations

### Mock Proofs (Current)

**Issue**: Current proofs are mock implementations, not cryptographically sound.

**Impact**: High for production - Not secure for mainnet

**Status**: Noir circuits planned for v0.2.0

**Recommendation**: Use testnet only until real proofs available.

### Memory Safety

**Issue**: JavaScript cannot guarantee secure memory clearing.

**Impact**: Medium - Keys may persist in memory

**Mitigation**:
- Minimize key lifetime
- Rely on OS memory protections
- WebAssembly isolation (future)

**Recommendation**: Use hardware wallets for high-value operations.

### No Hardware Wallet Support

**Issue**: Hardware wallet signing not implemented.

**Impact**: Medium - Limits security for high-value users

**Status**: Planned for future release

**Recommendation**: Use software wallets with appropriate security practices.

### Single Proof Provider

**Issue**: Only MockProofProvider currently available.

**Impact**: Cannot use in production

**Status**: NoirProofProvider planned

**Recommendation**: Testing and development only.

## Cryptographic Limitations

### Quantum Vulnerability

**Issue**: secp256k1 is vulnerable to Shor's algorithm.

**Impact**: Long-term - Not immediate threat

**Mitigation**:
- Post-quantum migration path (research)
- Lattice-based alternatives (evaluation)

**Recommendation**: Monitor quantum computing developments.

### No Trusted Setup... But Mock Proofs

**Issue**: While Noir doesn't require trusted setup, mock proofs provide no security.

**Impact**: Security claim doesn't apply until real proofs

**Status**: Real proofs will use universal setup

### Fixed Generator Construction

**Issue**: NUMS generator H is deterministically derived.

**Impact**: None - This is correct behavior

**Note**: Generator H derivation is verifiable and secure.

## Protocol Limitations

### No Partial Fills

**Issue**: Intents must be filled completely or not at all.

**Impact**: Low - May reduce quote availability

**Status**: Partial fill support planned

**Recommendation**: Use smaller amounts for better fill rates.

### Limited Chain Support

**Issue**: Not all chains are supported equally.

**Supported**: Ethereum, Solana, NEAR, Zcash
**Limited**: Polygon, Arbitrum (via NEAR Intents)
**Not supported**: Bitcoin (limited functionality)

**Status**: Chain support expanding

### Single Auditor per Compliant Intent

**Issue**: COMPLIANT mode supports one designated auditor per intent.

**Impact**: Low - Most use cases have single auditor

**Mitigation**: Multi-key encryption supports multiple viewers

### No On-Chain Verification

**Issue**: ZK proofs are verified off-chain by solvers.

**Impact**: Medium - Requires trust in solver verification

**Status**: On-chain verification planned with real circuits

## Operational Limitations

### No Key Recovery

**Issue**: Lost private keys cannot be recovered.

**Impact**: High - Permanent fund loss possible

**Mitigation**: User must backup keys

**Recommendation**: Implement proper key backup procedures.

### Irreversible Transactions

**Issue**: Shielded transactions cannot be reversed.

**Impact**: Standard for blockchain - Not SIP-specific

**Recommendation**: Verify all details before confirming.

### Viewing Key Revocation Doesn't Hide Past

**Issue**: Revoking viewing key doesn't un-reveal previously viewed data.

**Impact**: Medium - Permanent disclosure once shared

**Recommendation**: Use Transaction Viewing Keys for minimal exposure.

## Performance Limitations

### Browser Proof Generation (Future)

**Issue**: Proof generation in browser may be slow.

**Expected**: 2-5 seconds per proof

**Mitigation**:
- Web Workers for non-blocking
- Progress indicators
- Server-side option for performance-sensitive apps

### Scanning Efficiency

**Issue**: Scanning for stealth addresses scales linearly with transactions.

**Impact**: Medium for high-volume recipients

**Mitigation**:
- View tags reduce work by 256x
- Background scanning (async)
- Incremental scanning from last known block

## Summary Table

| Limitation | Severity | Status | Timeline |
|------------|----------|--------|----------|
| Mock proofs | High | Planned | v0.2.0 |
| Timing correlation | Medium | Research | TBD |
| Amount inference | Medium | Planned | v0.2.0 |
| Memory safety | Medium | Accepted | N/A |
| Hardware wallets | Medium | Planned | v0.3.0 |
| Quantum vulnerability | Low (long-term) | Research | TBD |
| Partial fills | Low | Planned | v0.2.0 |
| On-chain verification | Medium | Planned | v0.2.0 |

## Reporting Issues

If you discover additional limitations or security issues:

1. **Security issues**: Email security@sip-protocol.org
2. **General issues**: GitHub Issues
3. **Discussions**: GitHub Discussions
