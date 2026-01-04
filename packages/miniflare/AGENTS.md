# packages/miniflare/AGENTS.md

Local Workers simulator powered by workerd runtime.

## Structure

```
miniflare/
├── src/
│   ├── index.ts           # Miniflare class - main orchestrator
│   ├── runtime/           # workerd process management
│   │   └── config/        # Cap'n Proto serialization
│   ├── plugins/           # Binding simulators (kv, r2, d1, do, cache, etc.)
│   ├── workers/           # Workers running inside workerd
│   │   ├── core/          # Entry worker + proxy DO
│   │   └── shared/        # MiniflareDurableObject base, router, SQL
│   ├── http/              # HTTP server utilities
│   └── shared/            # Logging, errors, types
├── test/                  # Tests (AVA, not Vitest)
└── worker-metafiles/      # Build artifacts
```

## Architecture

```
MiniflareOptions → validateOptions() → assembleConfig() → serializeConfig() → workerd stdin
```

**Plugin system:** Each plugin provides:

- Zod schema for validation
- `getBindings()` → `Worker_Binding[]`
- `getServices()` → `Service[]` for workerd
- `getNodeBindings()` → Node.js access

**Magic proxy:** `ProxyClient` (Node.js) ↔ HTTP ↔ `ProxyServer` DO (workerd)

## Key Abstractions

| Class                    | Location                              | Purpose                       |
| ------------------------ | ------------------------------------- | ----------------------------- |
| `Miniflare`              | `src/index.ts`                        | Main orchestrator, public API |
| `Runtime`                | `src/runtime/index.ts`                | workerd process lifecycle     |
| `MiniflareDurableObject` | `src/workers/shared/object.worker.ts` | Base for all simulator DOs    |
| `ProxyClient`            | `src/plugins/core/proxy/client.ts`    | Node.js binding access        |

## Testing

**Framework: AVA** (not Vitest)

```bash
pnpm --filter miniflare test:ci
```

**Test helpers:**

- `miniflareTest()` - Creates fixture with Miniflare instance
- `namespace()` - Prefixes keys for parallel test isolation
- `MiniflareDurableObjectControlStub` - Fake timers, SQL queries

**Control operations:** Tests can control internals via `cf.miniflare.controlOp`:

```typescript
await object.enableFakeTimers(1000);
await object.advanceTime(5000);
await object.sqlQuery("SELECT * FROM _mf_entries");
```

## File Naming

- `*.worker.ts` - Type-checked under `@cloudflare/workers-types`
- Other `.ts` - Type-checked under `@types/node`

## Adding New Binding

1. Create simulator worker in `src/workers/<binding>/`
2. Extend `MiniflareDurableObject`, use `@GET`/`@POST` decorators
3. Create plugin in `src/plugins/<binding>/index.ts`
4. Register in `src/plugins/index.ts`
5. Add `Miniflare#get*()` method if needed
6. Write tests using `miniflareTest()` helper

## Conventions

- Use `this.db` and `this.blob` in DO workers for storage
- Router decorators: `@GET`, `@POST`, `@PUT`, `@DELETE`
- Plugins validate options with Zod schemas
- Config serialized to Cap'n Proto for workerd stdin

## ESLint Relaxations (Migration)

Many rules temporarily relaxed: `curly: off`, `no-explicit-any: off`, `no-floating-promises: off`
