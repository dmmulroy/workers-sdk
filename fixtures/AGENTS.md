# AGENTS.md - fixtures/

**Generated:** 2026-01-07 | **Commit:** fada563c1 | **Branch:** main

## Overview

75 test fixtures: workspace members with package.json, runnable via turbo. Real Worker projects for integration/E2E testing. Each has wrangler config (jsonc), many have vitest.config.mts. Examples: d1-worker-app, durable-objects-app, kv-app, pages-app, workflow, workers-with-assets.

## Structure

```
fixtures/
├── <fixture-name>/
│   ├── package.json           # Workspace member, named @fixture/<name>
│   ├── wrangler.jsonc         # Worker config (most use jsonc)
│   ├── vitest.config.mts      # 46/75 have custom config
│   ├── src/                   # Worker source
│   ├── tests/                 # Tests (*.test.ts)
│   └── tsconfig.json          # TS config
└── vitest-pool-workers-examples/  # 30+ examples of testing patterns
```

## Conventions

- **Naming**: `@fixture/<name>` in package.json
- **Scripts**: `test:ci` (vitest run), `test:watch` (vitest), `check:type` (tsc)
- **Deps**: wrangler from workspace, vitest/typescript from catalog
- **Config inheritance**: vitest.config.mts merges `vitest.shared` from root
- **Test files**: `tests/*.test.ts` (NOT `*.spec.ts`)
- **Wrangler config**: `.jsonc` format (not .toml)

## Common Patterns

| Fixture Type  | Examples                                         | Purpose                   |
| ------------- | ------------------------------------------------ | ------------------------- |
| Binding tests | d1-worker-app, kv-app, durable-objects-app       | Test Worker bindings      |
| Feature tests | workflow, workers-with-assets, email-worker      | Test specific features    |
| Tool tests    | vitest-pool-workers-examples, get-platform-proxy | Test dev tools            |
| Multi-worker  | dev-registry, dynamic-worker-loading             | Test service bindings/RPC |
| Edge cases    | wildcard-modules, unsafe-external-plugin         | Test uncommon scenarios   |

**vitest-pool-workers-examples**: 30+ subdirs demonstrating testing patterns (unit, integration, mocking, bindings). See its README for catalog.

**dev-registry**: Tests multi-worker dev scenarios (service bindings, registries).

**Tests run in turbo pipeline**: `pnpm test:ci` from root runs all fixture tests.
