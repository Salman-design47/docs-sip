# docs-sip

> Official SIP Protocol Documentation — https://docs.sip-protocol.org

---

## Overview

This repository contains the documentation for **SIP Protocol** (Shielded Intents Protocol), a privacy layer for cross-chain transactions.

## Tech Stack

- **Framework:** [Astro](https://astro.build/) + [Starlight](https://starlight.astro.build/)
- **Deployment:** Cloudflare Pages
- **Domain:** docs.sip-protocol.org

## Structure (Planned)

```
docs-sip/
├── src/
│   ├── content/
│   │   ├── docs/
│   │   │   ├── getting-started/
│   │   │   ├── guides/
│   │   │   ├── api/
│   │   │   └── concepts/
│   │   └── config.ts
│   └── pages/
├── public/
├── astro.config.mjs
└── package.json
```

## Content Sections

| Section | Description |
|---------|-------------|
| **Getting Started** | Installation, quick start, first intent |
| **Guides** | Privacy levels, viewing keys, compliance |
| **API Reference** | SDK methods, types, interfaces |
| **Concepts** | Stealth addresses, commitments, proofs |

## Development

```bash
# Install dependencies
pnpm install

# Start dev server
pnpm dev

# Build for production
pnpm build
```

## Related

- [sip-protocol](https://github.com/sip-protocol/sip-protocol) - Core SDK
- [sip-protocol.org](https://sip-protocol.org) - Marketing site

---

*Part of the [SIP Protocol](https://github.com/sip-protocol) ecosystem*
