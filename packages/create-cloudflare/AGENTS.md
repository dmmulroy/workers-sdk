# AGENTS.md - create-cloudflare

**Generated:** 2026-01-07 | **Commit:** fada563c1 | **Branch:** main

## Overview

C3: Interactive CLI for scaffolding Cloudflare projects. Generates Workers/Pages from 30+ templates (frameworks, hello-worlds, bindings examples). Handles package.json setup, git init, wrangler config, deployment flow.

## Structure

```
create-cloudflare/
├── src/
│   ├── cli.ts              # Entry: arg parsing → context → template flow
│   ├── templates.ts        # TemplateConfig registry (30+ templates)
│   ├── types.ts            # C3Context, C3Args, TemplateConfig
│   ├── workers.ts          # Workers-specific setup (types, wrangler config)
│   ├── pages.ts            # Pages-specific setup
│   ├── deploy.ts           # Deployment flow (offer → run → open browser)
│   ├── git.ts              # Git init/commit helpers
│   ├── dialog.ts           # Interactive prompts/summary
│   ├── metrics.ts          # Telemetry (Sparrow)
│   ├── validators.ts       # Project dir, template URL validation
│   ├── frameworks/         # Framework-specific config (index.ts)
│   ├── helpers/            # CLI utils, package managers, args, files
│   └── wrangler/           # Wrangler config generation, account selection
└── templates/              # 30+ template dirs (degit sources or file trees)
```

## Where to Look

| Task                    | Location                         | Notes                         |
| ----------------------- | -------------------------------- | ----------------------------- |
| Add template            | `src/templates.ts`               | Register in `templateMap`     |
| Modify template flow    | `src/cli.ts` line 44+            | main() orchestrates all steps |
| Framework CLI versions  | `src/frameworks/package.json`    | Dependabot-managed            |
| Template file structure | `templates/<template-name>/`     | Copied via degit or cp        |
| Wrangler config gen     | `src/wrangler/config.ts`         | updateWranglerConfig()        |
| Account selection       | `src/wrangler/accounts.ts`       | Interactive account picker    |
| Deploy logic            | `src/deploy.ts`                  | offerToDeploy() → runDeploy() |
| Git initialization      | `src/git.ts`                     | offerGit() → gitCommit()      |
| Arg parsing             | `src/helpers/args.ts`            | cliDefinition + parseArgs()   |
| Package manager detect  | `src/helpers/packageManagers.ts` | npm/pnpm/yarn/bun detection   |
