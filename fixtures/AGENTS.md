# AGENTS.md

**Generated:** 2026-01-07 | **Commit:** fada563c1 | **Branch:** main

## Overview

75+ test fixtures: full working projects validating Wrangler/Miniflare/vitest-pool. All `@fixture/*` workspace packages, `private: true`.

## Naming

```
worker-{feature}-app              # Workers: basic, logging, pubsub
durable-objects-app               # DO testing
pages-{functions|workerjs}-*-app  # Pages variants
workers-with-assets-{variant}     # Assets: spa, static-routing, run-worker-first
vitest-pool-workers-examples      # 30+ sub-fixtures for pool testing
{feature}-app                     # Misc: d1-worker-app, email-worker
```

## Structure

```
{fixture}/
├── package.json              # @fixture/* scope, workspace:* deps
├── wrangler.jsonc            # JSONC preferred (comments allowed)
├── tsconfig.json             # Extends @cloudflare/workers-tsconfig
├── vitest.config.mts         # Merges ../../vitest.shared
├── src/                      # Worker code, entrypoints
└── tests/                    # Integration tests (long-lived server)
```

vitest-pool-workers-examples uses `test/` instead of `tests/`.

## Testing Patterns

### Long-Lived Server (Standard)

```ts
import { runWranglerDev } from "@fixture/shared/run-wrangler-long-lived";

const { ip, port, stop } = await runWranglerDev(__dirname, ["--port=0"]);
await fetch(`http://${ip}:${port}`);
await stop();
```

### vitest-pool (Runtime Testing)

```ts
// vitest.config.ts
import { defineWorkersProject } from "@cloudflare/vitest-pool-workers/config";
// test/fetch-integration-self.test.ts
import { SELF } from "cloudflare:test";

export default defineWorkersProject({
	/* miniflare/wrangler options */
});

const response = await SELF.fetch("http://example.com");
```

## Adding New

1. Create `fixtures/{name}-app/`
2. `package.json`: `@fixture/{name}`, `private: true`, `workspace:*` deps
3. Add to `pnpm-workspace.yaml` if not matching glob
4. `wrangler.jsonc`: `main` or `entrypoints`, bindings
5. `vitest.config.mts`: Merge `../../vitest.shared`
6. Tests: `tests/` for HTTP, `test/` for vitest-pool
7. Run `pnpm install`, verify `pnpm --filter @fixture/{name} test:ci`
