---
title: Changelog
description: Release history for SIP Protocol SDK
---

# Changelog

Release history for `@sip-protocol/sdk`. All notable changes are documented here.

For detailed commit history, see the [GitHub releases](https://github.com/sip-protocol/sip-protocol/releases).

---

## Latest Release

### v0.1.1 — NEAR Intents NEP-141 Format

**Released:** November 29, 2025

[GitHub Release](https://github.com/sip-protocol/sip-protocol/releases/tag/v0.1.1) | [npm](https://www.npmjs.com/package/@sip-protocol/sdk/v/0.1.1)

#### Changes

- **fix(near):** Updated NEAR Intents message format to NEP-141 standard
- **fix(sdk):** Handle refund addresses for input chain type
- **fix(sdk):** Reject stealth addresses for non-EVM output chains
- **feat(stealth):** Add `publicKeyToEthAddress` for EVM chain support

#### Migration

No breaking changes. Update via:

```bash
npm update @sip-protocol/sdk
```

---

## Previous Releases

### v0.1.0 — Initial Release

**Released:** November 27, 2025

[GitHub Release](https://github.com/sip-protocol/sip-protocol/releases/tag/v0.1.0) | [npm](https://www.npmjs.com/package/@sip-protocol/sdk/v/0.1.0)

#### Features

- **Stealth Addresses**: EIP-5564 style one-time addresses (secp256k1)
- **Pedersen Commitments**: Homomorphic amount hiding
- **Viewing Keys**: Hierarchical key derivation for compliance
- **Privacy Levels**: TRANSPARENT, SHIELDED, COMPLIANT
- **NEAR Intents Adapter**: Integration with NEAR 1Click API
- **Wallet Adapters**: Solana (Phantom), Ethereum (MetaMask)
- **Zcash Integration**: RPC client and ShieldedService

#### Included Packages

- `@sip-protocol/sdk` — Core SDK
- `@sip-protocol/types` — TypeScript types

#### Installation

```bash
npm install @sip-protocol/sdk
```

#### Quick Start

```typescript
import { SIP, PrivacyLevel } from '@sip-protocol/sdk'

const sip = new SIP({ network: 'mainnet' })

const intent = await sip.createIntent({
  input: { chain: 'ethereum', token: 'ETH', amount: '1.0' },
  output: { chain: 'solana', token: 'SOL' },
  privacy: PrivacyLevel.SHIELDED,
})

const quotes = await sip.getQuotes(intent)
await sip.execute(intent, quotes[0])
```

---

## Upcoming

### v0.2.0 (Planned)

Expected features:

- **Multi-curve stealth addresses**: Ed25519 support for Solana/NEAR (already in dev branch)
- **Noir proof provider**: ZK proofs via Noir circuits
- **WASM build**: Browser-compatible proof generation
- **Enhanced wallet adapters**: Hardware wallet support

Track progress on [GitHub Milestones](https://github.com/sip-protocol/sip-protocol/milestones).

---

## Versioning

SIP Protocol follows [Semantic Versioning](https://semver.org/):

- **MAJOR** (x.0.0): Breaking changes to public API
- **MINOR** (0.x.0): New features, backward compatible
- **PATCH** (0.0.x): Bug fixes, backward compatible

### Stability

- `v0.x.x`: Development phase, API may change
- `v1.0.0`: Stable API (planned post-audit)

---

## Update Notifications

Stay informed about new releases:

- **Watch** the [GitHub repo](https://github.com/sip-protocol/sip-protocol)
- **Follow** [@rz1989sol](https://x.com/rz1989sol) on Twitter/X
- **npm**: `npm outdated @sip-protocol/sdk`

---

## Release Process

SIP Protocol releases follow this process:

1. **Development** on `dev` branch
2. **Testing** with 745+ automated tests
3. **Review** via pull request
4. **Release** to npm with GitHub tag
5. **Changelog** updated in docs

All releases are signed and published from CI/CD.

---

## Historical Notes

### Pre-Release Development (Nov 2025)

Before v0.1.0, SIP Protocol was developed through several milestones:

| Milestone | Focus | Commits |
|-----------|-------|---------|
| M1-M5 | Foundation, specs, core crypto | ~150 |
| M6 | Launch preparation, npm publish | ~30 |
| M7 | Demo integration, real testing | ~50 |
| M8 | Production hardening (ongoing) | ~40+ |

Key development highlights:

- **Nov 24**: Stealth address implementation complete
- **Nov 25**: Pedersen commitments with homomorphic properties
- **Nov 26**: NEAR Intents adapter integration
- **Nov 27**: Initial v0.1.0 release
- **Nov 29**: v0.1.1 with NEP-141 format fix

---

## Links

- [GitHub Releases](https://github.com/sip-protocol/sip-protocol/releases)
- [npm Package](https://www.npmjs.com/package/@sip-protocol/sdk)
- [Roadmap](/roadmap/)
- [GitHub Issues](https://github.com/sip-protocol/sip-protocol/issues)
