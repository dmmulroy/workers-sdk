# AGENTS.md

**Generated:** 2026-01-07 | **Scale:** 774+ TS files, ~184k LOC, 42 src dirs

## Overview

Workers CLI: deploy, dev server, config management, asset handling, D1/R2/KV/Vectorize/Queues/Hyperdrive, secrets, tail, init scaffolding.

## Structure

```
src/
├── commands/                  # CommandRegistry tree (NOT standard yargs)
├── api/                       # performApiFetch → fetchInternal → fetchResult/List/Graphql
├── dev/                       # DevEnv (event emitter), controllers, middleware loader facade
├── deployment-bundle/         # bundle.ts (574 LOC), entry resolution, esbuild orchestration
├── config/                    # wrangler.toml parsing via workers-utils
├── dev-registry/              # Worker registry, HMR, proxy routing
├── paths.ts                   # getBasePath() - replaces __dirname/__filename
├── user.ts                    # Auth, API token management (1450 LOC)
└── deploy.ts                  # Core deploy logic (1506 LOC)
```

### Command System

- **NOT yargs standard**: Custom tree via `createCommand()`, aliases with cycle detection
- **Metadata-driven**: `status`, `owner`, `deprecated`, `hidden`, `internal`
- **Definition**: `Command<Args>` with `positionalArgs`, `args()`, `handler()`
- **Registration**: `CommandRegistry.registerAll()` at startup

### Controllers (Event-Driven)

- **DevEnv**: Central bus, emits `reloadStart`, `reloadComplete`, `bundleStart`, `bundleComplete`
- **BundlerController**: esbuild orchestration, middleware loader facade (dynamic, NOT N passes)
- **ConfigController**: wrangler.toml watching, validation
- **LocalRuntimeController**: workerd/miniflare lifecycle
- **RemoteRuntimeController**: remote dev session management
- **ProxyController**: proxy ↔ local/remote routing

### Entry Resolution Order

```
--script → config.main → config.site['entry-point'] → --assets
```

### esbuild Plugins (11 total)

```
esbuild-plugins/
├── hybrid-nodejs-compat.ts    # Node.js compat injection
├── nodejs-compat-in-esm.ts    # ESM node: prefix handling
├── resolve-entry.ts           # Entry point resolution
├── middleware-loader.ts       # Dynamic facade generation
└── ...
```

## Where to Look

| Task                      | Location                                  |
| ------------------------- | ----------------------------------------- |
| Add CLI command           | `src/commands/*/` + `CommandRegistry`     |
| Modify deploy logic       | `src/deploy.ts:1-1506`                    |
| Add dev controller        | `src/dev/` + subscribe to `DevEnv` events |
| Change bundling           | `src/deployment-bundle/bundle.ts:1-574`   |
| Add esbuild plugin        | `src/deployment-bundle/esbuild-plugins/`  |
| API client logic          | `src/api/` (use `performApiFetch`)        |
| Auth/token management     | `src/user.ts:1-1450`                      |
| Middleware transformation | `src/middleware/` + middleware-loader     |
| Template embedding        | `templates/` + `tsup.config.ts` plugin    |

## Conventions

- **No console.\***: Use `logger` from `@cloudflare/cli` (ESLint: `no-console: error` except tests/scripts)
- **No globals**: `__dirname`/`__filename` → `getBasePath()`, `fetch` → `undici.fetch`
- **Config parsing**: Use `@cloudflare/workers-utils` (readConfig, diagnostics)
- **UI primitives**: Use `@cloudflare/cli` (spinner, select, confirm, TextInput)
- **Public unstable API**: `unstable_dev`, `DevEnv`, `startDevWorker`, `getPlatformProxy`
- **Imports**: `import type { X }` enforced, `node:` prefix required

## Anti-Patterns

- **Standard yargs**: Use `createCommand()`, register via `CommandRegistry`
- **Multiple esbuild passes**: Use middleware loader facade (dynamic generation)
- **Direct API fetch**: Use `performApiFetch` → preserves error handling/retries
- **Hardcoded paths**: Use `getBasePath()` for package-relative paths
- **console.log**: Use `logger.log()` / `.warn()` / `.error()`
- **Global fetch**: Import `{ fetch } from "undici"` explicitly

## Complexity Hotspots

- `deploy.ts` (1506 LOC): Asset uploads, service bindings, migrations, rollback
- `user.ts` (1450 LOC): OAuth, API tokens, CF_API_TOKEN/KEY, refresh logic
- `bundle.ts` (574 LOC): Entry resolution, esbuild config, plugin ordering, source maps
