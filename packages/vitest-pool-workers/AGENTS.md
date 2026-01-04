# packages/vitest-pool-workers/AGENTS.md

Vitest custom pool running tests inside workerd runtime.

## Structure

```
vitest-pool-workers/
├── src/
│   ├── pool/              # Node.js side - pool implementation
│   │   ├── index.ts       # ProcessPool interface
│   │   ├── config.ts      # Zod config schema
│   │   ├── loopback.ts    # Storage isolation (SQLite stack)
│   │   └── module-fallback.ts  # Vite module resolution
│   └── worker/            # workerd side - test executor
│       ├── index.ts       # Runner DO entry point
│       ├── lib/cloudflare/
│       │   ├── test.ts    # cloudflare:test public API
│       │   └── test-runner.ts  # Custom Vitest runner
│       ├── env.ts, events.ts, fetch-mock.ts, durable-objects.ts, ...
└── test/                  # Tests
```

## Architecture

```
Node.js (Pool)                    workerd (Worker)
┌────────────┐                    ┌──────────────────────────────────────┐
│ Vitest     │ ──WebSocket──────► │ Runner DO                            │
│ Pool       │                    │ ├── VitestExecutor (captured)        │
│            │ ◄──birpc RPC────── │ ├── WorkersTestRunner                │
└────────────┘                    │ └── User tests (Vite SSR transformed)│
      │                           └──────────────────────────────────────┘
      ▼
┌────────────┐
│ Miniflare  │
└────────────┘
```

**Execution modes:**

- `singleWorker: true` → Single Miniflare, single runner
- `singleWorker: false, isolatedStorage: false` → Single Miniflare, multiple runners
- `singleWorker: false, isolatedStorage: true` → Miniflare per test file

## `cloudflare:test` Module

```typescript
import {
	applyD1Migrations,
	createExecutionContext,
	createMessageBatch,
	createPagesEventContext,
	createScheduledController,
	env,
	fetchMock,
	getQueueResult,
	listDurableObjectIds,
	runDurableObjectAlarm,
	runInDurableObject,
	SELF,
	waitOnExecutionContext,
} from "cloudflare:test";
```

## Key Abstractions

| Abstraction             | Purpose                                       |
| ----------------------- | --------------------------------------------- |
| **Runner DO**           | Singleton ephemeral DO hosting test execution |
| **Stacked storage**     | Per-test SQLite isolation via file stack      |
| **Entrypoint wrappers** | Proxy classes for `SELF`, DO stubs            |
| **Module fallback**     | Vite SSR resolution for workerd               |
| **Fetch mock**          | Monkeypatched `fetch()` via undici MockAgent  |

## Config

```typescript
import { defineWorkersProject } from "@cloudflare/vitest-pool-workers/config";

export default defineWorkersProject({
	test: {
		poolOptions: {
			workers: {
				singleWorker: true, // or false
				isolatedStorage: true, // per-test isolation
				miniflare: {
					/* Miniflare options */
				},
				wrangler: { configPath: "./wrangler.jsonc" },
			},
		},
	},
});
```

## Test Patterns

**Unit test (direct call):**

```typescript
import {
	createExecutionContext,
	env,
	waitOnExecutionContext,
} from "cloudflare:test";
import worker from "../src";

test("fetch handler", async () => {
	const ctx = createExecutionContext();
	const response = await worker.fetch(new Request("http://x"), env, ctx);
	await waitOnExecutionContext(ctx);
	expect(response.status).toBe(200);
});
```

**Integration test (via SELF):**

```typescript
import { SELF } from "cloudflare:test";

test("via binding", async () => {
	const response = await SELF.fetch("http://example.com/api");
	expect(response.status).toBe(200);
});
```

**Durable Object test:**

```typescript
import { env, runInDurableObject } from "cloudflare:test";

test("DO state", async () => {
	const id = env.MY_DO.idFromName("test");
	const stub = env.MY_DO.get(id);

	const value = await runInDurableObject(stub, async (instance) => {
		return instance.ctx.storage.get("key");
	});
});
```

## Conventions

- Naming: `*-unit.test.ts`, `*-integration-self.test.ts`, `*-integration-auxiliary.test.ts`
- Use `runInDurableObject` to execute code in DO's I/O context
- Storage isolation happens at suite/test boundaries via `WorkersTestRunner`
- Compatibility flags forced: `nodejs_compat_v2`, `unsafe_module`, etc.
