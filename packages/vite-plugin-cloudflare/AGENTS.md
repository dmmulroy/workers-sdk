# AGENTS.md

**Generated:** 2026-01-07 | **Package:** @cloudflare/vite-plugin

## Overview

Vite Environment API integration: each Worker = separate Vite environment with custom runtime in workerd.

## Architecture

### Environment System

- `CloudflareDevEnvironment` extends `vite.DevEnvironment`: custom dev env per worker
- Each worker gets isolated Vite environment with module runner, WebSocket HMR, dep optimization
- `createCloudflareEnvironmentOptions()`: resolve conditions `["workerd", "worker", "module", "browser"]`, target `es2024`, `noExternal: true` for SSR pre-bundling

### Internal Workers

- **router-worker**: Routes requests to correct worker (re-exports `@cloudflare/workers-shared/router-worker`)
- **asset-worker**: Serves Vite dev server assets, handles HMR
- **runner-worker**: Executes user code via Vite module runner, establishes WebSocket for HMR
- **vite-proxy-worker**: Proxies requests between environments

### Runner Architecture

- `runner-worker/index.ts`: Wraps user exports in proxy classes (`WorkerEntrypoint`, `DurableObject`, `WorkflowEntrypoint`)
- `module-runner.ts`: Vite module runner integration, evaluates user code in workerd
- WebSocket channel: bidirectional HMR communication between Vite and workerd
- Proxy wrappers enable RPC by delegating property access to user code

### Plugin Composition (14 sub-plugins)

```
index.ts returns array:
  configPlugin           # Vite config setup, environment registration
  devPlugin              # Dev server initialization, Miniflare lifecycle
  previewPlugin          # Preview mode
  shortcutsPlugin        # CLI shortcuts (r: restart)
  debugPlugin            # Debug info output
  triggerHandlersPlugin  # Manual trigger handlers (queue, scheduled)
  virtualModulesPlugin   # virtual:cloudflare/*
  virtualClientFallbackPlugin
  outputConfigPlugin     # Build output config
  wasmHelperPlugin       # .wasm?module handling
  additionalModulesPlugin # .wasm?module, .bin, .txt → __CLOUDFLARE_MODULE__ placeholders
  nodeJsAlsPlugin        # Node.js AsyncLocalStorage polyfill
  nodeJsCompatPlugin     # nodejs_compat unenv polyfills
  nodeJsCompatWarningsPlugin
```

### Virtual Modules

- `virtual:cloudflare/worker-entry`: Injected entry wraps user entry, handles HMR accepts
- `virtual:cloudflare/export-types`: Runtime export introspection (WorkerEntrypoint, DurableObject, etc.)
- `virtual:cloudflare/user-entry`: Resolves to user's `main` field

### Additional Modules

- `.wasm?module`, `.bin`, `.txt`: Transformed to `__CLOUDFLARE_MODULE__` placeholders at build, runtime module loading

## Configuration

### Three-Layer Merge

1. **Defaults**: `compatibility_date = getLocalWorkerdCompatibilityDate()`, `name = dirname`
2. **wrangler.jsonc**: Standard Wrangler config
3. **Plugin config**: `cloudflare({ config: {...} })` option

### Auxiliary Workers

```ts
cloudflare({
	auxiliaryWorkers: [
		{ configPath: "./worker-a/wrangler.jsonc" },
		{ config: (cfg, { entryWorkerConfig }) => ({ ...cfg, name: "worker-b" }) },
	],
});
```

Config function receives `entryWorkerConfig` for cross-worker coordination.

### Non-Applicable Configs

**Replaced by Vite**: `alias` → `resolve.alias`, `define`, `minify` → `build.minify`  
**Not relevant**: `base_dir`, `build`, `find_additional_modules`, `no_bundle`, `preserve_file_names`, `rules`, `site`, `tsconfig`

### Remote Bindings

- `remoteBindings: true`: Use real CF resources (KV, R2, D1, etc.) instead of Miniflare stubs
- Session map per `configPath`: reuses sessions across dev server restarts
- Requires CF credentials

### Container Support

- Detects `base_image` in wrangler config → builds Docker container at dev server start
- Uses `@cloudflare/containers-shared` package

## Where to Look

| Task                    | Location                                               |
| ----------------------- | ------------------------------------------------------ |
| Add plugin              | `src/plugins/*.ts`, register in `index.ts`             |
| Modify environment opts | `src/cloudflare-environment.ts`                        |
| Runner wrapper logic    | `src/workers/runner-worker/index.ts`                   |
| Virtual module content  | `src/plugins/virtual-modules.ts`                       |
| Config resolution       | `src/plugin-config.ts`                                 |
| Worker config parsing   | `src/workers-configs.ts`                               |
| Miniflare integration   | `src/plugins/dev.ts`                                   |
| Remote bindings setup   | `src/miniflare-options.ts`                             |
| Container build         | `src/containers.ts`                                    |
| Node.js compat          | `src/nodejs-compat.ts`, `src/plugins/nodejs-compat.ts` |

## Testing

### Unit Tests

- `src/__tests__/`: Vitest, config resolution, validation logic

### E2E/Playground

- `e2e/`: Real Vite projects, mock npm registry
- `playground/`: 43 examples (basic, Pages, Hono, Remix, tRPC, WebSockets, DOs)
- Build: `tsdown.config.ts` with 5 targets (plugin + 4 internal workers)
