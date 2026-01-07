# AGENTS.md - packages/wrangler

**Primary CLI tool**: `wrangler dev`, `wrangler deploy`, 40+ subcommands for Workers/Pages/Platform products

## Structure

```
src/
├── core/                       # Command registry (tree-based, aliases)
│   ├── CommandRegistry.ts      # Core registration system
│   ├── register-yargs-command.ts
│   └── types.ts
├── api/startDevWorker/         # DevEnv controller pattern (event bus)
│   ├── DevEnv.ts               # Main orchestrator
│   ├── ConfigController.ts     # Config watching/merging
│   ├── BundlerController.ts    # Bundle watching
│   ├── LocalRuntimeController.ts   # Miniflare integration
│   ├── RemoteRuntimeController.ts  # Cloudflare preview
│   ├── ProxyController.ts      # Request routing
│   └── events.ts               # Event bus types
├── deploy/
│   └── deploy.ts               # 1500+ lines, main deploy logic
├── deployment-bundle/          # Bundling system
│   ├── bundle.ts               # Main bundler orchestration
│   ├── apply-middleware.ts     # Facade generation
│   ├── module-collection.ts    # Module graph
│   ├── esbuild-plugins/        # Custom esbuild plugins
│   └── rules.ts                # Module type rules
├── config/                     # Three-layer config system
│   ├── index.ts                # Raw -> Normalized -> Env-merged
│   ├── dot-env.ts              # .env handling
│   └── case-insensitive-env.ts
├── d1/                         # D1 commands
├── kv/                         # KV commands
├── r2/                         # R2 commands
├── queues/                     # Queues commands
├── pages/                      # Pages commands
├── vectorize/                  # Vectorize commands
├── workflows/                  # Workflows commands
├── vpc/                        # VPC commands
├── cloudchamber/               # Cloudchamber commands
├── pipelines/                  # Pipelines commands
└── [30+ other product dirs]
```

## Where to Look

| Task                   | Location                                 | Notes                         |
| ---------------------- | ---------------------------------------- | ----------------------------- |
| Add command            | `src/index.ts` + `src/<cmd>/`            | Register in main index        |
| Fix dev server         | `src/api/startDevWorker/`                | Controller pattern, event bus |
| Fix deploy             | `src/deploy/deploy.ts`                   | 1500+ lines, complex          |
| Fix bundling           | `src/deployment-bundle/`                 | Middleware facade, esbuild    |
| Fix config loading     | `src/config/index.ts`                    | Three-layer normalization     |
| Add esbuild plugin     | `src/deployment-bundle/esbuild-plugins/` | Many custom plugins           |
| Fix product subcommand | `src/<product>/`                         | 40+ product directories       |

## Conventions (Wrangler-Specific)

- `getBasePath()` instead of `__dirname` (works bundled/unbundled)
- `undici.fetch()` instead of global `fetch` (explicit import)
- `logger` from `@cloudflare/cli` instead of `console.log` in prod code
- `worker:` import protocol for embedded templates (e.g. `worker:middleware-loader`)
- Output to `wrangler-dist/` (not `dist/`)
- `/` for module paths (Windows compat via `toUrlPath()`)

## Anti-Patterns (Wrangler-Specific)

| Pattern        | Why                     | Instead                          |
| -------------- | ----------------------- | -------------------------------- |
| `__dirname`    | Breaks bundled builds   | `getBasePath()`                  |
| Global `fetch` | Wrangler standard       | `import { fetch } from "undici"` |
| `console.log`  | Prod code standard      | `logger` from cli                |
| `\` in paths   | Windows compat required | `toUrlPath()`, `/`               |
