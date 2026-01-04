# AGENTS.md

**Generated:** 2026-01-03 | **Commit:** 9e360f691 | **Branch:** main

## Overview

Cloudflare Workers SDK monorepo: CLI tools + libraries for Workers dev/test/deploy.

**Core tools:** Wrangler (CLI), Miniflare (local simulator), C3 (scaffolding), Vite plugin

## Structure

```
workers-sdk/
├── packages/
│   ├── wrangler/              # Main CLI → AGENTS.md
│   ├── miniflare/             # Local workerd simulator → AGENTS.md
│   ├── create-cloudflare/     # C3 scaffolding → AGENTS.md
│   ├── vite-plugin-cloudflare/ # Vite integration → AGENTS.md
│   ├── vitest-pool-workers/   # Test in workerd → AGENTS.md
│   ├── workers-utils/         # Config types/validation → AGENTS.md
│   ├── workers-shared/        # Asset + router workers (deployed to CF) → AGENTS.md
│   ├── cli/                   # CLI SDK (prompts, logging) → AGENTS.md
│   ├── pages-shared/          # Pages asset serving
│   ├── workflows-shared/      # Workflow engine (local dev only)
│   ├── containers-shared/     # Container API client
│   └── [dev-tools]/           # chrome-devtools-patches, quick-edit, workers-playground
├── fixtures/                  # 74 test fixtures → AGENTS.md
└── tools/                     # Build scripts, e2e helpers, deployment
```

## Commands

```bash
pnpm install                           # Install all deps
pnpm build                             # Build all (Turbo cached)
pnpm check                             # Lint + type + format
pnpm fix                               # Auto-fix lint + format
pnpm test:ci                           # Unit tests
pnpm test:e2e                          # E2E (needs CF creds)
pnpm test -F <pkg> "pattern"           # Single test by pattern
pnpm --filter <pkg> test:watch         # Watch mode
```

## Code Style

| Rule                | Enforcement                                  |
| ------------------- | -------------------------------------------- |
| `import type {}`    | `@typescript-eslint/consistent-type-imports` |
| No `any`            | `@typescript-eslint/no-explicit-any`         |
| No `!`              | `@typescript-eslint/no-non-null-assertion`   |
| Await/void promises | `@typescript-eslint/no-floating-promises`    |
| Curly braces always | `curly: error`                               |
| `node:` prefix      | `import/enforce-node-protocol-usage`         |
| Unused vars         | Prefix with `_`                              |
| No `.only()`        | `no-only-tests/no-only-tests`                |
| Prettier            | Tabs, double quotes, trailing comma es5      |

**Import order:** Built-ins → Third-party → Parent dirs → Siblings → Index → Types

## Testing

| Type             | Location                                 | Framework           |
| ---------------- | ---------------------------------------- | ------------------- |
| Unit             | `packages/*/src/__tests__/`              | Vitest              |
| Unit (miniflare) | `packages/miniflare/test/`               | AVA                 |
| Fixtures         | `/fixtures/*/tests/`                     | Vitest              |
| E2E              | `packages/*/e2e/`                        | Vitest              |
| Workers runtime  | `fixtures/vitest-pool-workers-examples/` | vitest-pool-workers |

**Shared config:** `vitest.shared.ts` - 50s timeout, `retry: 2` (flaky fixture tests)

## Package Relationships

```
wrangler
├── workers-utils (config types, validation)
├── miniflare (local dev)
├── workers-shared (asset serving)
├── pages-shared (Pages functionality)
├── cli (prompts, logging)
└── containers-shared (Docker utils)

vite-plugin-cloudflare
├── workers-utils (config types)
├── miniflare (local dev)
└── workers-shared (asset serving)

vitest-pool-workers
├── miniflare (test runner host)
└── workflows-shared (introspection)
```

## Changesets

**Every published package change needs a changeset.** See `.changeset/README.md`.

```bash
pnpm changeset                         # Create changeset
```

Exceptions: Add `no-changeset-required` label to PR.

## Anti-Patterns

- **Service environments** - Deprecated, don't use
- **`site.entry-point`** - Deprecated v1 config, use `main`
- **npm/yarn** - Use pnpm only
- **Committing to main** - Branch first

## PR Guidelines

- Title: `[package name] description`
- Use template from `.github/pull_request_template.md`
- Keep all checkboxes (don't delete unchecked)
- Remove "Fixes #..." if no issue

## Version Coupling

`miniflare` minor version = `workerd` minor version (enforced by `.github/changeset-version.js`)

## Deployment

- **npm packages:** Changesets → auto-release
- **Workers (workers-shared):** `tools/deployments/deploy-non-npm-packages.ts`
- **Prereleases:** `pkg-pr-new` on PRs (opt-in via `"workers-sdk": { "prerelease": true }`)
