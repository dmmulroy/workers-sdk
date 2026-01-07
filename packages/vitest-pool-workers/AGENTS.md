# AGENTS.md

**Generated:** 2026-01-07 | **Package:** @cloudflare/vitest-pool-workers

## Overview

Custom Vitest pool: tests execute INSIDE workerd runtime via WebSocket RPC to Runner Durable Object.

## Architecture

```
Node (Pool)  ←WebSocket RPC→  Runner DO (workerd)  ←import()→  User Code
├─ WorkersPool                ├─ __VITEST_POOL_WORKERS_RUNNER_DURABLE_OBJECT__
├─ Miniflare instance(s)      ├─ VitestExecutor (Vite transform pipeline)
├─ Module Fallback Service    ├─ Entrypoint/DO wrappers (lazy-load via executor)
└─ Loopback Service           └─ Custom test runner (stacked storage hooks)
```

**Pool (Node)**: `src/pool/index.ts` implements Vitest `ProcessPool`. Creates Miniflare, fetches Runner DO with `Upgrade: websocket` header, establishes birpc over WebSocket. Calls `runTests()` per spec group.

**Runner DO**: `src/worker/index.ts` singleton `__VITEST_POOL_WORKERS_RUNNER_DURABLE_OBJECT__`. Accepts WebSocket, monkeypatches `VitestExecutor.prototype.resolveUrl` to capture singleton, runs Vitest worker code (`vitest/dist/worker.js`) inside workerd.

**Module Fallback Service**: `src/pool/module-fallback.ts` bound via `unsafeModuleFallbackService`. Resolves/transforms modules workerd can't load. Handles CJS→ESM shimming (named export inference via `cjs-module-lexer`), Vite virtual modules, module rules (force type via `?mf_vitest_force=CompiledWasm`).

**Loopback Service**: `src/pool/loopback.ts` bound as `__VITEST_POOL_WORKERS_LOOPBACK_SERVICE`. Handles stacked storage push/pop (aborts all DOs, snapshots plugin state to disk), snapshot file I/O, D1 migrations.

**Chunking Socket**: `src/shared/chunking-socket.ts` wraps WebSocket. Splits messages >1MB into binary chunks followed by empty string marker.

**Cross-context execution**: `runInRunnerObject()` (src/worker/durable-objects.ts) runs code in Runner DO I/O context. `runInDurableObject()` runs in user DO context. Both use custom `cf` header with action ID, store results in `Map`.

**Entrypoint/DO Wrappers**: `src/worker/entrypoints.ts`, `src/worker/durable-objects.ts`. Proxy classes with intercepted prototypes. Lazy-load user code via `importModule()` → `executor.executeId()`. Enables `env.SELF.fetch()`, `runWithExportedHandler()`, isolated `getMockAgent()` per-instance.

## Storage Isolation

**Stacked Storage**: On-disk SQLite backups per plugin (KV, R2, D1, DO, Cache, Workflows). Push = copy current state → backup path. Pop = restore from backup, delete backup file.

- **Single Worker × Isolated**: Single Miniflare, push before each test, pop after
- **Single Worker × Shared**: Single Miniflare, no push/pop
- **Multi Worker × Isolated**: One Miniflare per test file, no push/pop (fresh instance)
- **Multi Worker × Shared**: One Miniflare, one runner worker per test file, shared storage

Triggered by custom test runner (`src/worker/lib/cloudflare/test-runner.ts`) calling loopback service via `beforeEach`/`afterEach`.

## Where to Look

| Task                           | Location                                      |
| ------------------------------ | --------------------------------------------- |
| Pool implementation            | `src/pool/index.ts:WorkersPool`               |
| Runner DO                      | `src/worker/index.ts:__VITEST_POOL_WORKERS_*` |
| Module resolution/CJS shimming | `src/pool/module-fallback.ts`                 |
| Storage stack push/pop         | `src/pool/loopback.ts`                        |
| Test runner with storage hooks | `src/worker/lib/cloudflare/test-runner.ts`    |
| Entrypoint/DO wrappers         | `src/worker/entrypoints.ts`                   |
| Cross-context helpers          | `src/worker/durable-objects.ts`               |
| User-facing API                | `src/worker/lib/cloudflare/test.ts`           |
| Config helper                  | `src/config/index.ts:defineWorkersProject`    |
| WebSocket message chunking     | `src/shared/chunking-socket.ts`               |

## Conventions

**Required compat flags**: `nodejs_compat_v2` (forced), `unsafe_module` (injected), `export_commonjs_default` (validated). Extra features: `nodejs_tty_module`, `nodejs_fs_module`, `nodejs_http_modules`, `nodejs_perf_hooks_module`.

**Unsafe APIs**: `UnsafeEval` (bound as `__VITEST_POOL_WORKERS_UNSAFE_EVAL`), `unsafeModuleFallbackService`, `unsafeStickyBlobs`, `abortAllDurableObjects()`.

**Service bindings**: `__VITEST_POOL_WORKERS_SELF_SERVICE` (kCurrentWorker), `__VITEST_POOL_WORKERS_LOOPBACK_SERVICE`, `__VITEST_POOL_WORKERS_RUNNER_OBJECT`.

**Virtual modules**: `__VITEST_POOL_WORKERS_USER_OBJECT` (entrypoint/DO wrappers), `__VITEST_POOL_WORKERS_DEFINES` (define injection, like Vite).

**Worker naming**: Runner workers prefixed `__vitest_pool_workers_runner-{projectName}[-{testFileHash}-{testFile}]`.

**Persistence cleanup**: `scheduleStorageReset()` after tests (background), so results display immediately while `.wrangler/state` dirs emptied.
