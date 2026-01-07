# AGENTS.md

**Generated:** 2026-01-07 | **Commit:** fada563c1 | **Branch:** main

## Overview

Cloudflare Workers SDK monorepo: CLI tools + libraries for Workers development. Core: Wrangler (CLI), Miniflare (local simulator), C3 (scaffolding), Vite plugin.

## Structure

```
workers-sdk/
├── packages/
│   ├── wrangler/              # Main CLI (see packages/wrangler/AGENTS.md)
│   ├── miniflare/             # Local workerd simulator (see AGENTS.md)
│   ├── vite-plugin-cloudflare/# Vite integration (see AGENTS.md)
│   ├── create-cloudflare/     # C3 scaffolding (see AGENTS.md)
│   ├── vitest-pool-workers/   # Test in workerd (see AGENTS.md)
│   ├── workers-shared/        # Shared: asset-worker + router-worker
│   ├── pages-shared/          # Pages asset serving
│   ├── workers-utils/         # Config parsing, errors, types
│   ├── cli/                   # Prompts, spinners, UI primitives
│   └── ...                    # 20+ more packages
├── fixtures/                  # 75+ integration test apps (see AGENTS.md)
├── tools/                     # CI scripts, deployments, validation
└── turbo.json                 # Build orchestration
```

## Commands

```bash
pnpm install                    # Install all deps (NEVER npm/yarn)
pnpm build                      # Build all (Turbo cached)
pnpm check                      # Lint + typecheck + format
pnpm fix                        # Auto-fix lint/format
pnpm test:ci                    # Run tests
pnpm test -F wrangler "pattern" # Single test by name
pnpm --filter <pkg> test:watch  # Watch mode
```

## Code Style (Enforced)

| Rule                    | Enforcement                                |
| ----------------------- | ------------------------------------------ |
| Tabs not spaces         | `.editorconfig`, Prettier                  |
| `import type { X }`     | ESLint `consistent-type-imports`           |
| No `any`                | `@typescript-eslint/no-explicit-any`       |
| No `!` assertions       | `@typescript-eslint/no-non-null-assertion` |
| Await all promises      | `@typescript-eslint/no-floating-promises`  |
| Always `{}` braces      | `curly: error`                             |
| `node:` prefix          | `import/enforce-node-protocol-usage`       |
| Unused vars: `_` prefix | `@typescript-eslint/no-unused-vars`        |
| No `.only()` in tests   | `no-only-tests/no-only-tests`              |

## Anti-Patterns (This Project)

- **Service environments**: DEPRECATED, will be removed
- **`build.upload`**: Legacy v1 - use `main` field
- **Global `fetch`**: Use undici in Wrangler (restricted global)
- **`__dirname`/`__filename`**: Use `getBasePath()` in Wrangler
- **Major version bumps**: Forbidden for wrangler (changeset validation)
- **Workers deploy packages**: Currently blocked - manual deploy only

## Build System

- **Turbo**: Remote cache with signatures, task dependencies
- **pnpm catalogs**: Centralized versions (`catalog:default`)
- **TypeScript**: `bundler` moduleResolution, `alwaysStrict: false`
- **Vitest**: `pool: forks`, 15-50s timeouts, `retry: 2` for flaky tests

## Testing

| Type        | Location                    | Pattern                |
| ----------- | --------------------------- | ---------------------- |
| Unit        | `packages/*/src/__tests__/` | Vitest, MSW mocks      |
| Integration | `fixtures/*/tests/`         | Long-lived dev servers |
| E2E         | `packages/*/e2e/`           | Real CF credentials    |
| Runtime     | vitest-pool-workers         | Tests run IN workerd   |

## CI/Deployment

- **Changesets**: Required for all published package changes
- **Prerelease**: `pkg-pr-new` publishes PR packages
- **Worker deploys**: `workers-shared` uses `versions upload` with git tags
- **Hotfix**: Manual workflow bypasses changesets
- **External PRs**: Team member triggers CI via proxy workflow

## Key Patterns

### Workspace Dependencies

```json
"wrangler": "workspace:*",
"@cloudflare/workers-tsconfig": "workspace:^",
"vitest": "catalog:default"
```

### Miniflare Version Sync

Custom script forces Miniflare minor = workerd minor version. Prevents drift.

### Package Metadata

```json
"workers-sdk": { "prerelease": true, "deploy": true }
```

Controls prerelease publishing and worker deployment behavior.

## Where to Look

| Task                 | Location                                     |
| -------------------- | -------------------------------------------- |
| Add Wrangler command | `packages/wrangler/src/` + `CommandRegistry` |
| Add binding type     | `packages/miniflare/src/plugins/`            |
| Add C3 template      | `packages/create-cloudflare/templates/`      |
| Add Vite feature     | `packages/vite-plugin-cloudflare/src/`       |
| Test Workers runtime | `packages/vitest-pool-workers/`              |
| Integration test     | `fixtures/` (create new fixture)             |

## Gotchas

1. **Fixtures are workspace packages**: Listed in `pnpm-workspace.yaml`, must be `private: true`
2. **Tools is a package**: Has own tests, linting, type-checking
3. **Nested workspaces**: vite-plugin playgrounds are separate workspace members
4. **Config validation**: `lint-turbo.mjs` validates turbo.json ↔ vitest configs match
5. **Windows CI**: 50s test timeouts, smart-tabs for mixed indentation

## Subdirectory AGENTS.md

- `packages/wrangler/AGENTS.md` - CLI architecture, command system
- `packages/miniflare/AGENTS.md` - Plugin system, workerd integration
- `packages/vite-plugin-cloudflare/AGENTS.md` - Vite environments, HMR
- `packages/create-cloudflare/AGENTS.md` - Template system, c3.ts
- `packages/vitest-pool-workers/AGENTS.md` - Custom pool, storage isolation
- `fixtures/AGENTS.md` - Test fixture patterns
