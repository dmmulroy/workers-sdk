# packages/wrangler/AGENTS.md

Main CLI for Workers development and deployment.

## Structure

```
wrangler/
├── src/
│   ├── index.ts           # API exports (startWorker, getPlatformProxy)
│   ├── cli.ts             # CLI entry, main() runs when require.main
│   ├── core/              # Command registry system
│   ├── api/               # Public APIs + DevEnv controller architecture
│   │   └── startDevWorker/  # Modern dev API (event-driven controllers)
│   ├── config/            # Wrangler config reading (delegates to workers-utils)
│   ├── cfetch/            # Authenticated CF API client
│   ├── user/              # Auth, login, requireAuth
│   ├── dev/               # wrangler dev implementation
│   ├── deploy/            # wrangler deploy implementation
│   ├── d1/, r2/, kv/      # Feature-specific commands
│   └── utils/             # Shared utilities
├── e2e/                   # E2E tests (real CF API)
├── templates/             # Runtime templates (24 items)
└── wrangler-dist/         # Build output (not dist/)
```

## Command System

Commands use tree-based registry via `createCommand()` / `createNamespace()`:

```typescript
export const d1CreateCommand = createCommand({
	metadata: { description, owner: "Team: X", status: "stable" },
	args: {
		/* yargs options */
	},
	positionalArgs: ["name"],
	handler: (args, ctx) => {
		/* ctx has: config, logger, fetchResult, sdk */
	},
});
```

**Registration:** `index.ts` → `createCLIParser()` → `registry.define([...])` → yargs

## DevEnv Architecture (Event-Driven)

`src/api/startDevWorker/` uses controller bus pattern:

```
DevEnv
├── ConfigController      # Config parsing/updates
├── BundlerController     # esbuild bundling
├── RuntimeController[]   # Local/Remote runtime
└── ProxyController       # HTTP + inspector proxy
```

**Events:** `configUpdate` → `bundleStart` → `bundleComplete` → `reloadStart` → `reloadComplete`

## Key Files

| File                               | Purpose                                     |
| ---------------------------------- | ------------------------------------------- |
| `src/core/CommandRegistry.ts`      | Command tree + yargs integration            |
| `src/core/create-command.ts`       | `createCommand()`, `createNamespace()`      |
| `src/api/startDevWorker/DevEnv.ts` | Event bus + controller orchestration        |
| `src/cfetch/internal.ts`           | `fetchResult()` - auth'd CF API calls       |
| `src/config/index.ts`              | `readConfig()` - delegates to workers-utils |

## Testing

```bash
pnpm --filter wrangler test:ci         # Unit tests (pool: forks)
pnpm --filter wrangler test:e2e        # E2E tests (needs CF creds)
```

**Unit tests:** `src/__tests__/` - Heavy mocking (msw, prompts, undici)  
**E2E tests:** `e2e/` - Real CF API, long timeouts

## Conventions

- **Handler context:** `{ config, logger, fetchResult, sdk, errors }` injected
- **API vs CLI:** `src/api/` for programmatic, commands for CLI
- **Config flow:** CLI args → `readConfig()` → `normalizeAndValidateConfig()` → Config
- **Auth flow:** `cfetch/` → `requireLoggedIn()` → `requireApiToken()` → CF API

## Anti-Patterns

- Don't import from `wrangler` in vite-plugin - use namespace import
- Don't use `__dirname`/`__filename`/`fetch` globals (eslint restricted)
- Don't use `console` directly - use `logger`
