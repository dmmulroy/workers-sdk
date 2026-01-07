# packages/miniflare/AGENTS.md

**Generated:** 2026-01-07 | **Parent:** ../../AGENTS.md

## Overview

Local Workers simulator powered by workerd runtime. Plugin architecture: 29 plugins, one per binding type. Bindings implemented as Workers running IN workerd (not Node). Storage via TypedSql + Blob abstractions in workers. Router pattern with decorators. Magic proxy for Node.js binding access.

## Structure

```
miniflare/
├── src/
│   ├── index.ts               # Main Miniflare class (2700+ lines)
│   ├── plugins/               # 29 binding plugins (kv, r2, do, d1, cache, queues, etc)
│   │   ├── core/              # Core services, proxy, inspector, errors
│   │   │   └── proxy/         # Magic proxy for Node.js binding access
│   │   └── shared/            # Plugin base, persistence, routing
│   ├── workers/               # 44 .worker.ts files (run IN workerd, not Node)
│   │   ├── shared/            # MiniflareDurableObject base, TypedSql, Blob, router
│   │   ├── kv/                # KV namespace.worker.ts
│   │   ├── cache/             # Cache entry.worker.ts
│   │   └── ...                # One per binding type
│   ├── runtime/               # workerd process mgmt + Cap'n Proto config generation
│   │   └── config/            # serializeConfig() - generates workerd config
│   ├── http.ts                # Fetch implementations, request/response handling
│   └── cf.ts                  # cf object setup
├── test/                      # *.spec.ts convention
└── dist/src/                  # Non-standard nested output
```

## Where to Look

| Task                    | Location                           | Notes                            |
| ----------------------- | ---------------------------------- | -------------------------------- |
| Add binding type        | `src/plugins/{name}/index.ts`      | New plugin + worker              |
| Implement binding       | `src/workers/{name}/*.worker.ts`   | Runs IN workerd                  |
| Storage primitives      | `src/workers/shared/sql.worker.ts` | TypedSql abstraction             |
| Router/HTTP handling    | `src/workers/shared/router.ts`     | @GET/@POST/@PUT/@DELETE          |
| Base class for bindings | `src/workers/shared/object.ts`     | MiniflareDurableObject           |
| workerd config          | `src/runtime/config/`              | Cap'n Proto serialization        |
| Magic proxy             | `src/plugins/core/proxy/`          | Node.js <-> workerd bridge       |
| Main orchestration      | `src/index.ts`                     | Miniflare class, 2700+ lines     |
| Plugin registration     | `src/plugins/index.ts`             | PLUGIN_ENTRIES                   |
| Fix simulator behavior  | Relevant plugin or worker file     | Likely in src/workers/{binding}/ |

## Conventions

- `.worker.ts` files run IN workerd (not Node) - use Workers APIs only
- `.ts` files run in Node - can use Node APIs
- Router decorators: `@GET`, `@POST`, `@PUT`, `@DELETE` on `RouteHandler` methods
- `MiniflareDurableObject` base class for all binding simulators
- `KeyValueStorage` interface for SQL-backed storage
- Plugin exports: `options`, `sharedOptions`, `getBindings`, `getServices`, `getNodeBindings`
- Worker imports: `worker:shared/index`, `worker:{plugin}/{name}`
- Cap'n Proto config via `serializeConfig()` for workerd
- Persist paths: `.mf/` default, configurable via `{binding}Persist` options
- Service names: `{PLUGIN_NAME}:ns:{namespace}` pattern

## Anti-Patterns

| Pattern                   | Why                        | Instead                              |
| ------------------------- | -------------------------- | ------------------------------------ |
| Node APIs in .worker.ts   | Runs in workerd, not Node  | Use Workers APIs                     |
| Workers APIs in .ts       | Runs in Node               | Use Node APIs or proxy               |
| Direct workerd access     | Abstracted by runtime/     | Use Runtime class + Config types     |
| Manual Cap'n Proto        | Generated + serialized     | Use runtime/config/workerd.ts types  |
| Global state in workers   | Isolate-local, not process | Use Durable Objects for shared state |
| Synchronous storage ops   | All async in workerd       | Await all storage operations         |
| console.log in production | No logging infra in worker | Use bindings.log or throw errors     |
| Direct SQL                | Abstraction exists         | Use KeyValueStorage interface        |
| `any` in worker types     | Type safety critical       | Proper typing                        |
| Importing plugins in core | Circular deps              | Use SERVICE_LOOPBACK pattern         |
