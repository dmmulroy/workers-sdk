# fixtures/AGENTS.md

74 test fixture packages for integration testing.

## Organization

Each fixture is a standalone workspace package:

```
fixtures/<name>/
├── package.json       # @fixture/<name>
├── wrangler.jsonc     # Worker config
├── vitest.config.mts  # Test config
├── src/               # Source files
└── tests/             # Test files
```

## Naming Conventions

| Pattern                 | Purpose                        |
| ----------------------- | ------------------------------ |
| `*-app`                 | Complete Worker applications   |
| `pages-*`               | Cloudflare Pages testing       |
| `workers-with-*`        | Workers with specific features |
| `*-tests`               | Specific test scenarios        |
| `vitest-pool-workers-*` | Pool workers testing           |
| `*-example`             | Example/demo projects          |

## Package → Fixture Mapping

| Package                  | Fixtures                                                                   |
| ------------------------ | -------------------------------------------------------------------------- |
| `wrangler`               | `worker-app`, `pages-*`, `d1-*`, `local-mode-tests`, `get-platform-proxy*` |
| `miniflare`              | `miniflare-node-test`, `start-worker-node-test`                            |
| `vitest-pool-workers`    | `vitest-pool-workers-examples/*` (25+ sub-projects)                        |
| `vite-plugin-cloudflare` | `dev-registry/`                                                            |
| `workers-shared`         | `workers-shared-asset-config/`                                             |

## Adding a Fixture

1. Create directory and `package.json`:

   ```json
   {
   	"name": "@fixture/my-feature",
   	"private": true,
   	"scripts": {
   		"check:type": "tsc",
   		"test:ci": "vitest run"
   	},
   	"devDependencies": {
   		"@cloudflare/workers-tsconfig": "workspace:*",
   		"@cloudflare/workers-types": "catalog:default",
   		"wrangler": "workspace:*",
   		"vitest": "catalog:default"
   	}
   }
   ```

2. Create `wrangler.jsonc`:

   ```jsonc
   {
   	"name": "my-feature-app",
   	"main": "src/index.ts",
   	"compatibility_date": "2024-01-01",
   }
   ```

3. Create `vitest.config.mts`:

   ```typescript
   import { defineConfig } from "vitest/config";

   export default defineConfig({
   	test: { include: ["tests/**/*.test.ts"] },
   });
   ```

4. Run: `pnpm install && pnpm test:ci -F @fixture/my-feature`

## For vitest-pool-workers Testing

Use `defineWorkersProject` and add to `vitest-pool-workers-examples/`:

```typescript
import { defineWorkersProject } from "@cloudflare/vitest-pool-workers/config";

export default defineWorkersProject({
	test: {
		poolOptions: {
			workers: { wrangler: { configPath: "./wrangler.jsonc" } },
		},
	},
});
```

## Shared Utilities

`fixtures/shared/` provides:

- `run-wrangler-long-lived.ts` - Helper for `wrangler dev` in tests

## Test Frameworks

- **Most fixtures:** Vitest (standard Node.js)
- **miniflare tests:** Node.js test runner (`node --test`)
- **Pool workers:** vitest-pool-workers (workerd runtime)
