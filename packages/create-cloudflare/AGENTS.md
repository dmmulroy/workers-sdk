# packages/create-cloudflare/AGENTS.md

Project scaffolding CLI (C3).

## Structure

```
create-cloudflare/
├── src/
│   ├── cli.ts             # Entry point, main flow
│   ├── templates.ts       # Template system core
│   ├── types.ts           # TemplateConfig, C3Context
│   ├── frameworks/        # Framework CLI integration
│   │   └── package.json   # Framework CLI versions (dependabot-managed)
│   └── helpers/           # Args, codemod, files, package managers
├── templates/             # Built-in templates
│   ├── hello-world*/      # Basic worker templates
│   ├── astro/, next/, nuxt/, svelte/, react/, vue/, hono/ ...  # Frameworks
│   └── common/, scheduled/, queues/, openapi/  # Other starters
└── e2e/                   # E2E tests
```

## Template System

**TemplateConfig interface:**

```typescript
{
  configVersion: 1,
  id: "my-template",
  displayName: "My Template",
  platform: "workers" | "pages",
  copyFiles?: { path: "./ts" } | { variants: { js, ts } },
  generate?: (ctx) => Promise<void>,     // Run framework CLI
  configure?: (ctx) => Promise<void>,    // Post-gen config
  transformPackageJson?: (pkg, ctx) => Promise<object>,
}
```

**Multi-platform frameworks:** Use `MultiPlatformTemplateConfig` with `platformVariants: { pages, workers }`

## CLI Flow

```
main() → parseArgs() → runCli()
  ├── createContext()      # Prompts + template selection
  ├── create()             # Generate project, copy files, npm install
  ├── configure()          # Post-gen: wrangler config, types, git
  ├── deploy()             # Optional deploy to CF
  └── printSummary()
```

## Adding New Template

1. Create `templates/<name>/c3.ts`:

   ```typescript
   export default {
   	configVersion: 1,
   	id: "my-template",
   	displayName: "My Template",
   	platform: "workers",
   	copyFiles: { variants: { js: { path: "./js" }, ts: { path: "./ts" } } },
   } satisfies TemplateConfig;
   ```

2. Create variant dirs: `templates/<name>/js/`, `templates/<name>/ts/`

3. For frameworks, add CLI version to `src/frameworks/package.json`

## Testing

```bash
pnpm --filter create-cloudflare test:ci    # Unit tests
pnpm --filter create-cloudflare test:e2e   # E2E tests
```

**E2E config:** `e2e/tests/frameworks/test-config.ts`, `e2e/tests/workers/test-config.ts`

**Filter tests:**

```bash
E2E_FRAMEWORK_TEMPLATE_TO_TEST=astro:workers pnpm test:e2e -F create-cloudflare -- frameworks
```

## Conventions

- Template IDs use kebab-case
- Package names: `@fixture/...` scope not used; use plain names
- Framework CLI versions pinned in `src/frameworks/package.json` (dependabot updates)
- Use `usesTypescript(ctx)` helper to detect TS projects
