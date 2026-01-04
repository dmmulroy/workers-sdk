# packages/wrangler/src/api/AGENTS.md

Public APIs for programmatic wrangler usage + DevEnv controller architecture.

## Structure

```
api/
├── index.ts              # Public exports (startWorker, deprecated unstable_dev)
├── startDevWorker/       # Modern dev API (event-driven)
│   ├── index.ts          # startWorker() entry
│   ├── DevEnv.ts         # Event bus + controller orchestration
│   ├── ConfigController.ts    # Config resolution
│   ├── BundlerController.ts   # esbuild bundling
│   ├── LocalRuntimeController.ts   # Miniflare runtime
│   ├── RemoteRuntimeController.ts  # Remote runtime
│   ├── ProxyController.ts     # HTTP + inspector proxy
│   ├── types.ts          # StartDevWorkerInput, Worker types
│   └── events.ts         # Event type definitions
└── integrations/
    └── platform/
        └── index.ts      # getPlatformProxy(), unstable_getMiniflareWorkerOptions
```

## DevEnv Architecture

Event-driven controller bus:

```
DevEnv
├── ConfigController      # Config parsing/updates → configUpdate events
├── BundlerController     # esbuild bundling → bundleStart/Complete events
├── RuntimeController[]   # Local/Remote → reloadStart/Complete events
└── ProxyController       # HTTP + inspector proxy → proxyData events
```

**Event flow:** `configUpdate` → `bundleStart` → `bundleComplete` → `reloadStart` → `reloadComplete`

## Key APIs

**`startWorker(options)`** - Start dev worker programmatically:

```typescript
const worker = await startWorker({
	config: "wrangler.jsonc",
	// or inline config
});
await worker.fetch("http://localhost/");
await worker.dispose();
```

**`getPlatformProxy()`** - Get Miniflare bindings for testing:

```typescript
const { env, cf, ctx } = await getPlatformProxy<Env>();
// env.MY_KV, env.MY_DO, etc.
```

## Conventions

- Controllers communicate via typed events only
- `DevEnv` manages controller lifecycle
- Proxy handles both HTTP and inspector WebSocket
- Remote runtime uses preview sessions (authenticated)
