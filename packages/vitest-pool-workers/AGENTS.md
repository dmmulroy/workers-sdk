# AGENTS.md - vitest-pool-workers

**Custom Vitest pool + config helpers for running tests IN workerd runtime (not Node).**

## Overview

Vitest pool implementation that executes tests inside Workers runtime via Miniflare. Supports unit + integration tests with Workers APIs, bindings, isolated storage, HTTP mocking. Uses Vite for bundling/transforms.

## Structure

```
src/
├── pool/               # Pool orchestration (Node-side)
│   ├── index.ts       # Pool entry, manages Miniflare instances
│   ├── config.ts      # WorkersPoolOptions schema + validation
│   └── loopback.ts    # Worker↔Vitest RPC bridge
├── worker/            # Test execution runtime (workerd-side)
│   ├── index.ts       # Test runner entry, patches globals
│   ├── lib/cloudflare/test.ts  # cloudflare:test exports (env, SELF, fetchMock, etc)
│   ├── fetch-mock.ts  # undici-based HTTP mocking
│   ├── env.ts         # Per-test binding injection
│   ├── durable-objects.ts  # runInDurableObject/runDurableObjectAlarm
│   └── workflows.ts   # Workflow introspection
├── config/
│   ├── index.ts       # defineWorkersProject/defineWorkersConfig helpers
│   ├── d1.ts          # applyD1Migrations helper
│   └── pages.ts       # createPagesEventContext helper
└── shared/            # Shared types/utils
```

## Where to Look

| Task                        | Location                            | Notes                                 |
| --------------------------- | ----------------------------------- | ------------------------------------- |
| Add pool option             | `src/pool/config.ts`                | WorkersPoolOptionsSchema (zod)        |
| Add cloudflare:test export  | `src/worker/lib/cloudflare/test.ts` | Re-exports from test-internal         |
| Fix binding access in tests | `src/worker/env.ts`                 | Per-test binding proxy                |
| Fix HTTP mocking            | `src/worker/fetch-mock.ts`          | undici MockAgent wrapper              |
| Fix Durable Object testing  | `src/worker/durable-objects.ts`     | runInDurableObject impl               |
| Add config helper           | `src/config/index.ts`               | Export alongside defineWorkersProject |
| Fix module resolution       | `src/pool/module-fallback.ts`       | Node builtin polyfills                |
| Debug test execution        | `src/worker/index.ts`               | Patches console, vm, etc              |

## Key Patterns

- **Config**: Use `defineWorkersProject()` in vitest.config.ts, returns plugin that injects `cloudflare:test` module
- **Pool options**: `test.poolOptions.workers` - main, isolatedStorage, singleWorker, miniflare, wrangler
- **Isolated storage**: `isolatedStorage: true` (default) - storage copied per-test, rolled back after
- **Single worker**: `singleWorker: true` - all tests serial in same worker, shared module cache (faster for many small files)
- **Main entry**: `main` option registers Worker for `SELF` binding + Durable Objects without scriptName
- **Bindings in tests**: Import `env` from `cloudflare:test` - proxy to current test's bindings
- **HTTP mocking**: `fetchMock` from `cloudflare:test` - declarative undici MockAgent interface
- **Wrangler config**: `wrangler: { configPath, environment }` - load bindings from wrangler.jsonc
- **Remote bindings**: `remoteBindings: true` - connect to remote resources with `remote: true` in wrangler config
- **Additional exports**: `additionalExports` - declare exports not inferred from source (virtual modules, wildcards)
- **D1 migrations**: `applyD1Migrations(env.DB, path)` helper from `@cloudflare/vitest-pool-workers/config`
- **RPC**: birpc + chunking-socket for Node↔workerd communication, serializes test results/errors
