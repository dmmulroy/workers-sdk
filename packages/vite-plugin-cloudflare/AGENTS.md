# AGENTS.md

**Package:** `@cloudflare/vite-plugin`

## Overview

Vite plugin for Cloudflare Workers dev. Returns array of plugins, not single plugin. Module Runner executes INSIDE workerd via WebSocket bridge to Vite dev server. Multi-worker architecture (router, asset, vite-proxy, user workers) orchestrated via single Miniflare instance. **Prerelease status** - API unstable.

## Structure

```
src/
├── index.ts                    # Main plugin export
├── plugin-config.ts            # Config resolution/validation
├── context.ts                  # PluginContext + SharedContext pattern
├── cloudflare-environment.ts   # CloudflareDevEnvironment (extends Vite DevEnvironment)
├── build.ts                    # Build via Vite Builder API
├── workers/
│   ├── runner-worker/          # Module Runner (runs IN workerd)
│   ├── router-worker/          # Routes requests to workers
│   ├── asset-worker/           # Serves Vite dev assets
│   └── vite-proxy-worker/      # Proxies to Vite
├── plugins/
│   ├── virtual-modules.ts      # virtual:cloudflare/worker-entry, virtual:cloudflare/export-types
│   ├── additional-modules.ts   # ?cloudflare-internal imports
│   ├── dev.ts                  # CloudflareDevEnvironment setup
│   ├── config.ts               # Vite config manipulation
│   ├── shortcuts.ts            # Dev server keyboard shortcuts
│   └── trigger-handlers.ts     # Cron/queue triggers
├── workers-configs.ts          # Multi-worker Miniflare config
├── miniflare-options.ts        # MiniflareOptions generation
├── websockets.ts               # WS bridge for Module Runner
└── export-types.ts             # Type generation
playground/                     # 40+ examples (workspace members)
e2e/                           # E2E tests
```

## Where to Look

| Task                          | Location                         | Notes                             |
| ----------------------------- | -------------------------------- | --------------------------------- |
| Add plugin hook               | `src/plugins/`                   | Multiple plugins compose          |
| Fix Module Runner             | `src/workers/runner-worker/`     | Executes IN workerd               |
| Fix multi-worker routing      | `src/workers/router-worker/`     | Routes to asset/user workers      |
| Add virtual module            | `src/plugins/virtual-modules.ts` | Prefix `virtual:cloudflare/`      |
| Fix dev environment           | `src/cloudflare-environment.ts`  | Extends Vite DevEnvironment       |
| Fix build                     | `src/build.ts`                   | Uses buildApp hook                |
| Fix Miniflare integration     | `src/miniflare-options.ts`       | Multi-worker config generation    |
| Add binding type              | `src/plugin-config.ts`           | Config schema + validation        |
| Fix type generation           | `src/export-types.ts`            | env.d.ts generation               |
| Test new feature              | `playground/`                    | Create new example (add tsconfig) |
| Fix wrangler.toml integration | `src/vite-config.ts`             | Config merging logic              |

## Conventions

- **Plugin returns array**: `cloudflare()` returns `Plugin[]` not single plugin
- **PluginContext pattern**: Shared state via `PluginContext`, persisted state via `SharedContext`
- **Virtual module prefix**: `virtual:cloudflare/` for all virtual modules
- **Additional modules**: `?cloudflare-internal` suffix for internal worker modules
- **Worker names**: `MAIN_ENTRY_NAME` constant for primary worker
- **Single Miniflare**: One instance, multi-worker config via `setOptions()`
- **WebSocket bridge**: Module Runner communicates to Vite via WS
- **Playground tests**: Each example is workspace member with package.json
- **Build tool**: tsdown (not tsup)
- **Prerelease**: API unstable, breaking changes allowed
