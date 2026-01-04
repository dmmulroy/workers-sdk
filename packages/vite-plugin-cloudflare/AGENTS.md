# packages/vite-plugin-cloudflare/AGENTS.md

Vite plugin for Cloudflare Workers development.

## Structure

```
vite-plugin-cloudflare/
├── src/
│   ├── index.ts           # Main entry, plugin composition
│   ├── context.ts         # PluginContext - shared state
│   ├── plugin-config.ts   # Config resolution
│   ├── cloudflare-environment.ts  # CloudflareDevEnvironment
│   ├── miniflare-options.ts       # Config → Miniflare options
│   ├── plugins/           # Internal Vite plugins (14+)
│   └── workers/           # Internal workers (runner, router, asset, vite-proxy)
├── playground/            # 41 example projects
│   ├── __test-utils__/    # Test utilities
│   └── <example>/         # Each example is a workspace package
└── e2e/                   # E2E tests
```

## Architecture

`cloudflare()` returns array of 14+ Vite plugins:

```
vite-plugin-cloudflare (coordinator)
vite-plugin-cloudflare:config
vite-plugin-cloudflare:dev
vite-plugin-cloudflare:preview
vite-plugin-cloudflare:shortcuts
vite-plugin-cloudflare:debug
vite-plugin-cloudflare:trigger-handlers
vite-plugin-cloudflare:virtual-modules
...
```

All plugins share `PluginContext` instance.

## Environment API

Each worker maps to a Vite environment:

- Entry worker → default environment
- Auxiliary workers → separate environments
- Worker names converted: `my-worker` → `my_worker` (dashes to underscores)

**CloudflareDevEnvironment** extends `vite.DevEnvironment`:

- `initRunner()` - Initialize module runner in Miniflare
- `fetchModule()` - Resolve modules via Vite

## Adding Playground Examples

1. Create `playground/<name>/`:

   ```
   package.json        # @playground/<name>
   vite.config.ts
   wrangler.jsonc
   src/index.ts
   __tests__/<name>.spec.ts
   ```

2. `package.json`:

   ```json
   {
   	"name": "@playground/<name>",
   	"private": true,
   	"devDependencies": {
   		"@cloudflare/vite-plugin": "workspace:*",
   		"wrangler": "workspace:*"
   	}
   }
   ```

3. `vite.config.ts`:

   ```typescript
   import { cloudflare } from "@cloudflare/vite-plugin";

   export default defineConfig({
   	plugins: [cloudflare({ inspectorPort: false, persistState: false })],
   });
   ```

4. Tests use `../../__test-utils__` for `page`, `viteTestUrl`, etc.

**Variants:** Create `vite.config.<variant>.ts` + `__tests__/<variant>/` directory

## Testing

```bash
pnpm --filter @cloudflare/vite-plugin test:ci           # Unit tests
pnpm --filter @cloudflare/vite-plugin test:e2e          # E2E tests
pnpm --filter @vite-plugin-cloudflare/playground test:ci # Playground tests
```

Playground tests run in both dev (`test:ci:serve`) and build (`test:ci:build`) modes.

## Conventions

- Don't use named imports from `"wrangler"` - use namespace import
- Test configs: `inspectorPort: false`, `persistState: false`
- Workers compile to `workers/*.js` in dist
- Build tool: `tsdown` (not tsup)
