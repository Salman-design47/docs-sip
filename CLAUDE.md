# CLAUDE.md - SIP Documentation

> **Ecosystem Hub:** See [sip-protocol/CLAUDE.md](https://github.com/sip-protocol/sip-protocol/blob/main/CLAUDE.md) for full ecosystem context

**Repository:** https://github.com/sip-protocol/docs-sip
**Purpose:** Official documentation website for SIP Protocol

---

## Quick Reference

**Tech Stack:** Astro 5, Starlight, MDX
**Deployment:** docs.sip-protocol.org (Docker + GHCR)

**Key Commands:**
```bash
npm install               # Install dependencies
npm run dev               # Dev server (localhost:4321)
npm run build             # Build for production
npm run preview           # Preview build
```

---

## Key Files

| Path | Description |
|------|-------------|
| `src/content/docs/` | Documentation pages (MDX) |
| `src/content/config.ts` | Content collections config |
| `astro.config.mjs` | Astro + Starlight configuration |
| `src/styles/` | Custom styles |
| `src/assets/` | Images and assets |
| `public/` | Static files |

---

## Content Structure

```
src/content/docs/
├── index.mdx           # Home page
├── getting-started/    # Quickstart guides
├── concepts/           # Core concepts (privacy, stealth, etc.)
├── sdk/                # SDK reference
├── api/                # API documentation
└── guides/             # How-to guides
```

---

## Repo-Specific Guidelines

**DO:**
- Use MDX for interactive documentation
- Include code examples with syntax highlighting
- Keep navigation structure shallow

**DON'T:**
- Duplicate content from SDK JSDoc
- Add custom components without need

---

## Starlight Features

- Automatic sidebar from file structure
- Built-in search
- Dark/light mode
- i18n ready

---

**Last Updated:** 2025-12-02
