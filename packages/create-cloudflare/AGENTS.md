# AGENTS.md

**Generated:** 2026-01-07 | **Commit:** fada563c1 | **Branch:** main

## OVERVIEW

C3: Interactive scaffolding wizard (create → configure → deploy) for Workers/Pages projects.

## TEMPLATE SYSTEM

### TemplateConfig Schema

```typescript
{
  configVersion: 1,
  id: string,
  displayName: string,
  platform: "workers" | "pages",
  frameworkCli?: string,           // Dispatches to create-astro, create-react, etc
  copyFiles?: {
    path: string                   // Single variant
    | variants: {                  // Multi-variant (js/ts/python)
        js: { path: "./js" },
        ts: { path: "./ts" },
        python: { path: "./py" }
      },
    selectVariant?: (ctx) => string,
    destinationDir?: string | ((ctx) => string)
  },
  generate?: (ctx) => Promise<void>,    // Runs framework CLI
  configure?: (ctx) => Promise<void>,   // Post-generation setup
  transformPackageJson?: (pkg) => Promise<object>,
  workersTypes?: "generated" | "installed" | "none"
}
```

### Multi-Platform Templates

```typescript
MultiPlatformTemplateConfig = {
	displayName: string,
	platformVariants: {
		pages: TemplateConfig,
		workers: TemplateConfig,
	},
};
```

49 c3.ts files across `templates/*`. Frameworks auto-select variant via `selectVariant` or default to `ctx.args.lang`.

## CODEMODS

- `parseFile(path)`: Parse JS/TS → recast AST (auto-detects parser)
- `transformFile(path, visitor)`: Apply visitor, write back
- `loadTemplateSnippets(ctx)`: Load `templates/*/snippets/*.ts` as AST nodes
- `mergeObjectProperties(obj, props)`: Deep-merge object literals in AST

Recast preserves formatting. Used by framework templates to inject bindings (e.g., qwik `vite.config.ts`).

## HELPERS

| Helper                   | Purpose                                                |
| ------------------------ | ------------------------------------------------------ |
| `readJSON/writeJSON`     | Comment-preserving JSON via `comment-json`             |
| `readToml/writeToml`     | TOML via `smol-toml`                                   |
| `usesTypescript(ctx)`    | Check `tsconfig.json` presence                         |
| `updateWranglerConfig`   | Replace `<WORKER_NAME>`, `<COMPATIBILITY_DATE>`        |
| `updatePackageName`      | Replace `<PACKAGE_NAME>`, `TBD` in pkg.json            |
| `addWranglerToGitIgnore` | Inject `.wrangler/`, `.dev.vars*`, `.env*`             |
| `copyTemplateFiles`      | Copy variant, rename `__dot__gitignore` → `.gitignore` |

## WHERE TO LOOK

| Task                  | Location                                          |
| --------------------- | ------------------------------------------------- |
| Add template          | `templates/<name>/c3.ts` + variant dirs           |
| Modify wizard flow    | `src/cli.ts` (main), `src/templates.ts` (context) |
| Add codemod helper    | `src/helpers/codemod.ts`                          |
| Framework dispatch    | `src/frameworks/`                                 |
| Wrangler config logic | `src/wrangler/config.ts`                          |
| Worker/Pages setup    | `src/workers.ts`, `src/pages.ts`                  |

## CONVENTIONS

- **File naming**: Build renames `.gitignore` → `__dot__gitignore` (reversed at copy)
- **Context threading**: `C3Context` passed to all hooks (args, project, template, deployment, account)
- **Placeholders**: `<WORKER_NAME>`, `<COMPATIBILITY_DATE>`, `<PACKAGE_NAME>`, `<PROJECT_NAME>` (Python)
- **Type generation**: Default `"generated"` (wrangler types), `"installed"` (@cloudflare/workers-types), `"none"` (static sites)
- **Framework templates**: Use `generate` hook + framework CLI, then `configure` for CF-specific setup
