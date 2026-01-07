# AGENTS.md

**Generated:** 2026-01-07 | **Commit:** fada563c1 | **Branch:** main

## Overview

Cloudflare Workers SDK monorepo: CLI tools + libraries for Workers development. Core: Wrangler (CLI), Miniflare (local simulator), C3 (scaffolding), Vite plugin.

## Structure

```
workers-sdk/
├── packages/
│   ├── wrangler/              # Main CLI (700+ files) - see packages/wrangler/AGENTS.md
│   ├── miniflare/             # Local workerd simulator - see packages/miniflare/AGENTS.md
│   ├── vite-plugin-cloudflare/ # Vite integration - see packages/vite-plugin-cloudflare/AGENTS.md
│   ├── vitest-pool-workers/   # Test in workerd runtime - see packages/vitest-pool-workers/AGENTS.md
│   ├── create-cloudflare/     # C3 scaffolding CLI - see packages/create-cloudflare/AGENTS.md
│   ├── workers-utils/         # Shared config/types/errors
│   ├── workers-shared/        # Internal Workers infra (DEPLOYED - careful!)
│   ├── pages-shared/          # Wrangler + Pages shared (DEPLOYED - careful!)
│   ├── workflows-shared/      # Workflows engine internals
│   ├── containers-shared/     # Cloudchamber API client
│   └── cli/                   # CLI rendering SDK
├── fixtures/                  # 75 test fixtures (workspace members) - see fixtures/AGENTS.md
└── tools/                     # CI/deployment scripts
```

## Commands

```bash
pnpm install                    # Install all deps (NEVER npm/yarn)
pnpm build                      # Build all (turbo cached)
pnpm check                      # Lint + type + format check
pnpm fix                        # Auto-fix lint/format
pnpm test:ci                    # Run tests
pnpm test:e2e                   # E2E (needs CF credentials)
pnpm -F <pkg> test:watch        # Watch mode for package
pnpm test -F <pkg> "pattern"    # Single test by name
```

## Where to Look

| Task                     | Location                                      | Notes                     |
| ------------------------ | --------------------------------------------- | ------------------------- |
| Add wrangler command     | `packages/wrangler/src/core/`                 | Command registry system   |
| Add binding type         | `packages/workers-utils/src/config/`          | Config + validation       |
| Simulate binding locally | `packages/miniflare/src/plugins/`             | Plugin per binding type   |
| Fix deploy issue         | `packages/wrangler/src/deploy/`               | deploy.ts is 1500+ lines  |
| Fix dev server           | `packages/wrangler/src/api/startDevWorker/`   | DevEnv controller pattern |
| Add C3 template          | `packages/create-cloudflare/src/templates.ts` | Template definitions      |
| Test in Workers runtime  | Use `vitest-pool-workers`                     | Tests run IN workerd      |

## Code Style (Enforced)

- `import type { X }` for type-only imports
- `node:` prefix for Node imports (`import fs from "node:fs"`)
- Curly braces always (`if (x) { ... }`)
- Await or void promises (no floating)
- Prefix unused vars with `_`
- No `.only()` in tests

## Anti-Patterns (This Project)

| Pattern                  | Why                | Instead                 |
| ------------------------ | ------------------ | ----------------------- |
| `any` type               | eslint error       | Proper typing           |
| Non-null assertion (`!`) | eslint error       | Type narrowing          |
| `__dirname`/`__filename` | Wrangler-specific  | `getBasePath()`         |
| Global `fetch`           | Wrangler-specific  | undici fetch            |
| `console.log`            | Wrangler prod code | Logger from cli package |
| npm/yarn                 | Monorepo standard  | pnpm only               |
| Service environments     | Deprecated         | Standard envs           |

## Changesets

**Every user-facing change requires a changeset** or `no-changeset-required` label.

```bash
pnpm changeset                  # Create changeset
```

- `patch`: Bugfixes, small improvements
- `minor`: New features, deprecations, experimental breaking changes
- `major`: Breaking stable changes (**forbidden for wrangler currently**)
- NO h1/h2/h3 headers in changeset descriptions (breaks changelog)

## Testing Strategy

| Type            | Framework           | Location                    | Notes                 |
| --------------- | ------------------- | --------------------------- | --------------------- |
| Unit tests      | Vitest              | `src/__tests__/` or `test/` | `.test.ts` files      |
| Miniflare tests | Vitest              | `test/*.spec.ts`            | `.spec.ts` convention |
| Workers runtime | vitest-pool-workers | Various                     | Runs IN workerd       |
| Fixtures        | Vitest              | `fixtures/*/`               | Real worker projects  |
| E2E             | Vitest              | `e2e/`                      | Needs CF credentials  |

## Build System

- **Turbo**: Orchestrates builds, caches aggressively
- **pnpm catalog**: Centralized versions (`workerd`, `esbuild`, `vitest`)
- **tsup**: Most packages use tsup for bundling
- Wrangler outputs to `wrangler-dist/` (non-standard)
- Miniflare outputs to `dist/src/` (nested, non-standard)

## Critical Paths (Review Carefully)

- `packages/workers-shared/` - Deployed to production Workers
- `packages/pages-shared/` - Deployed to production Pages
- `packages/workflows-shared/` - Workflows engine internals
- `.changeset/` config - Affects all releases

## PR Checklist

- [ ] Branch off main (not on main)
- [ ] Changeset added (or `no-changeset-required` label)
- [ ] Tests included (or `no-tests` label + justification)
- [ ] `pnpm check` passes
- [ ] PR title: `[package-name] description`

## Environment Variables

Tests may need:

- `TEST_CLOUDFLARE_API_TOKEN` - E2E tests
- `TEST_CLOUDFLARE_ACCOUNT_ID` - E2E tests
- `WRANGLER_LOG=debug` - Debug logging

## Gotchas

- Fixtures are **workspace members** - they have package.json and run in turbo
- `workers-shared` has **deploy scripts** - changes deploy to CF on release
- Miniflare v3 **removed CLI** - `npx miniflare` shows deprecation error
- Module paths must use `/` (not `\`) for Windows compat
- vitest-pool-workers tests run IN workerd, not Node
- 50s default test timeout (Windows CI compatibility)
