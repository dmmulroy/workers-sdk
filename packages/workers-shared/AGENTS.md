# packages/workers-shared/AGENTS.md

Asset + router workers deployed to Cloudflare infrastructure. Also bundled into Miniflare for local dev.

## Structure

```
workers-shared/
├── asset-worker/          # Serves static assets
│   ├── src/worker.ts      # WorkerEntrypoint class, RPC methods (unstable_*)
│   ├── wrangler.jsonc     # Production config with stable_id, param bindings
│   └── tests/
├── router-worker/         # Routes between user Worker and assets
│   ├── src/worker.ts      # Standard fetch handler, routing logic
│   ├── wrangler.jsonc     # Production config with internal bindings
│   └── tests/
├── utils/                 # Shared utilities (re-exported from index.ts)
│   ├── types.ts           # Zod schemas: AssetConfig, RouterConfig
│   ├── configuration/     # Parsers for _headers, _redirects
│   └── helpers.ts         # File utilities, content type detection
├── scripts/               # Deployment helpers
└── index.ts               # Re-exports utils only (not workers)
```

## Deployment

**Production (Cloudflare):**

- Deployed via `wrangler versions upload` (Gradual Deployments)
- Version tags: `aw-<version>` (asset), `rw-<version>` (router)
- Staging auto-deploys on `main` changes to `packages/workers-shared/**`
- Uses internal bindings: `param`, `internal_assets`, `origin`

**Local (Miniflare):**

- Workers re-exported into Miniflare:
  - `miniflare/src/workers/assets/assets.worker.ts`
  - `miniflare/src/workers/assets/router.worker.ts`
- Asset manifest uses `hash(path + mtime)` (not content) for perf

## Commands

```bash
pnpm --filter @cloudflare/workers-shared test:ci          # All tests
pnpm --filter @cloudflare/workers-shared deploy:staging   # Stage both
pnpm --filter @cloudflare/workers-shared deploy           # Prod (CI only)
```

## Key Differences: Local vs Production

| Aspect        | Local (Miniflare)    | Production             |
| ------------- | -------------------- | ---------------------- |
| Asset storage | File system          | KV (`internal_assets`) |
| Content hash  | `hash(path + mtime)` | `hash(content)`        |
| Config        | JSON bindings        | `param` bindings       |
| User Worker   | Service binding      | `origin` binding       |

## Config Types

- **RouterConfig:** `has_user_worker`, `invoke_user_worker_ahead_of_assets`, `static_routing`
- **AssetConfig:** `html_handling`, `not_found_handling`, `redirects`, `headers`

## Package Flags

```json
"workers-sdk": {
  "prerelease": true,   // Include in pkg-pr-new
  "deploy": true        // Non-npm deploy via deploy-non-npm-packages.ts
}
```
