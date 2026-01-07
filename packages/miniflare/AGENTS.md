# AGENTS.md

**Generated:** 2026-01-07 | **Package:** miniflare

## OVERVIEW

Local Workers runtime simulator: multi-process workerd orchestrator implementing CF platform via plugin architecture.

## ARCHITECTURE

### Core Components

- **Runtime**: `child_process.spawn()` workerd with Cap'n Proto config via stdin
- **Platform-as-Workers**: R2/KV/Cache/Queues/D1 implemented AS Workers in `src/workers/*.worker.ts`
- **ProxyClient**: Node↔workerd bridge via Durable Object "heap" + devalue serialization
- **BlobStore**: Content-addressable, 40-byte hex IDs, SQLite atomicity
- **DevRegistry**: File-based service discovery for multi-process coordination
- **Inspector Proxy**: Multiplexes single port → multiple workers by name

### Config Generation

- **NOT direct capnp**: TypeScript types (`runtime/config/workerd.ts`) → programmatic encoding via `capnp-es`
- **`serializeConfig()`**: Dynamic Cap'n Proto struct building from typed objects
- **Debug**: `MINIFLARE_WORKERD_CONFIG_DEBUG=path.json` dumps config

### Module Resolution

- **Acorn AST walking**: `worker-loader` plugin resolves `node:*`, `npm:`, `cloudflare:*` imports
- **Path aliases**: `miniflare:shared`, `miniflare:zod` → bundled platform workers
- **Worker imports**: `worker:shared/index` resolves to compiled `.worker.ts` files

## PLUGINS

28 plugins implement `Plugin<Options, SharedOptions>`:

- **Services**: `getServices()` returns workerd `Service[]` configs
- **Bindings**: `getBindings()` returns `Worker_Binding[]` arrays
- **Key plugins**: core (routing), do (namespaces), kv/r2/cache/d1/queues (platform workers), assets, workflows, hyperdrive

### External Services

- **Service bindings** to missing workers → dev registry proxy with fallback services
- **Outbound DO proxy**: `OUTBOUND_DO_PROXY_SERVICE_NAME` forwards to external sessions
- **Dev registry**: Filesystem-watched registry for cross-session service/DO bindings

## WHERE TO LOOK

| Task                  | Location                                       |
| --------------------- | ---------------------------------------------- |
| Add plugin            | `src/plugins/<name>/index.ts`                  |
| Platform worker       | `src/workers/<name>/*.worker.ts`               |
| Workerd config        | `src/runtime/config/index.ts:50` (serialize)   |
| Proxy system          | `src/plugins/core/proxy/client.ts`             |
| BlobStore             | `src/plugins/shared/blob.ts`                   |
| Dev registry          | `src/shared/dev-registry.ts`                   |
| Inspector proxy       | `src/plugins/core/inspector-proxy.ts`          |
| Module resolution     | `src/plugins/worker-loader/`                   |
| Live reload/WebSocket | `src/index.ts:1011-1031` (loopback)            |
| Instance registry     | `src/index.ts:890-897` (`_initialise…`)        |
| Version sync          | `scripts/deps.ts` (miniflare ↔ workerd minor) |

## CONVENTIONS

- **Tests**: `*.spec.ts` (NOT `.test.ts`), vitest
- **Path aliases**: `miniflare:shared` → `src/workers/shared/index.worker.ts`
- **DO storage**: In-memory (`#tmpPath`) default, opt-in persistent via `*Persist` options
- **BlobStore IDs**: 40-byte hex SHA-256, stored in `.mf/` or tmpPath
- **SQL schemas**: Defined in `*.worker.ts` files (e.g. `r2/schemas.worker.ts`)
