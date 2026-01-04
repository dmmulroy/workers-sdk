# packages/workers-utils/AGENTS.md

Shared config types, validation, parsing for wrangler + vite-plugin.

## Structure

```
workers-utils/
├── src/
│   ├── index.ts              # Barrel exports
│   ├── config/               # Config types + validation (~2700 LOC)
│   │   ├── config.ts         # Config, RawConfig types
│   │   ├── environment.ts    # Environment, inheritable fields
│   │   ├── validation.ts     # normalizeAndValidateConfig()
│   │   ├── validation-helpers.ts  # inheritable(), deprecated(), experimental()
│   │   └── diagnostics.ts    # Diagnostics class (structured errors/warnings)
│   ├── parse.ts              # parseTOML, parseJSON, parseJSONC, parsePackageJSON
│   ├── errors.ts             # UserError, FatalError, DeprecationError
│   ├── worker.ts             # CfWorkerInit, CfModule, binding types
│   └── compatibility-date.ts # getLocalWorkerdCompatibilityDate()
└── test-helpers/             # writeWranglerConfig, seed helpers
```

## Key Exports

**Config types:**

- `Config`, `RawConfig`, `Environment`, `RawEnvironment`
- `ConfigFields`, `ConfigBindingOptions`, `RedirectedRawConfig`

**Validation:**

- `normalizeAndValidateConfig()` - Main entry, returns `{ config, diagnostics }`
- `Diagnostics` class - Tree-structured error/warning collection
- `hasProperty()`, `isRequiredProperty()`, `isOptionalProperty()`

**Parsing:**

- `parseTOML()`, `parseJSON()`, `parseJSONC()`, `parsePackageJSON()`
- `ParseError`, `APIError`

**Errors:**

- `UserError`, `FatalError`, `DeprecationError`, `CommandLineArgsError`

**Utilities:**

- `getLocalWorkerdCompatibilityDate()` - Used by wrangler, vite-plugin, c3
- `resolveWranglerConfigPath()`, `findWranglerConfig()`
- `formatConfigSnippet()` - Format config as TOML/JSON

## Validation Patterns

**Inheritable vs Non-inheritable:**

```typescript
// Fields that inherit from top-level to named environments
inheritable({ ... })

// Fields that don't inherit
notInheritable({ ... })

// Deprecated fields (warns but allows)
deprecated({ ... }, "Use X instead")
```

**Diagnostics pattern:**

```typescript
const diagnostics = new Diagnostics();
diagnostics.errors.push({ message: "..." });
diagnostics.warnings.push({ message: "..." });
// Never throws - collects all issues
return { config, diagnostics };
```

## Consumers

| Package           | Usage                                                   |
| ----------------- | ------------------------------------------------------- |
| wrangler          | Heavy (~100 imports): config, validation, errors, types |
| vite-plugin       | Light: `getLocalWorkerdCompatibilityDate()` only        |
| create-cloudflare | `getLocalWorkerdCompatibilityDate()`                    |

## Conventions

- Validation never throws - returns Diagnostics
- Path normalization: relative → absolute based on config location
- Config format detection via file extension (.toml vs .json/.jsonc)
