# packages/cli/AGENTS.md

CLI SDK: prompts, logging, colors for wrangler + create-cloudflare.

## Structure

```
cli/
├── index.ts          # Main exports: logRaw, log, status, shapes, space
├── interactive.ts    # Prompts: textInput, spinner, confirm, select, multiSelect
├── select-list.ts    # SelectList + SelectListItem components
├── colors.ts         # brandColor, dim, gray, white, hidden + bg* colors
├── streams.ts        # stdout, stderr wrappers
├── cursor.ts         # Cursor control utilities
├── args.ts           # CLI argument parsing helpers
├── error.ts          # CancelError class
└── check-macos-version.ts  # macOS version detection
```

## Key Exports

**Logging:**

- `logRaw(msg)` - Print to stdout (respects log level)
- `log(msg)` - Stylized with bar prefix
- `status.error`, `status.warning`, `status.info`, `status.success`
- `setLogLevel()`, `getLogLevel()` - Global log level control

**Prompts:**

- `textInput({ question, defaultValue, validate })`
- `spinner({ message, start, stop })`
- `confirm({ question, defaultValue })`
- `select({ question, options })`
- `multiSelect({ question, options })`

**Visual:**

- `shapes.diamond`, `shapes.bar`, `shapes.corners.*`
- `brandColor()`, `dim()`, `gray()`, `white()`
- `space(n)` - Non-trimmable spaces

## Usage

```typescript
import { logRaw, spinner, status, textInput } from "@cloudflare/cli";

logRaw(`${status.info} Starting...`);

const s = spinner();
s.start("Working...");
// ...
s.stop("Done!");

const name = await textInput({
	question: "Project name?",
	defaultValue: "my-app",
});
```

## Consumers

| Package           | Usage                          |
| ----------------- | ------------------------------ |
| wrangler          | Logging, status badges         |
| create-cloudflare | All prompts, spinners, logging |
| miniflare         | Limited (has own logger)       |

## Conventions

- All output via `logRaw()` or `log()` - never `console.log`
- Log levels: `none`, `error`, `warn`, `info`, `log`, `debug`
- Chalk for colors (not kleur - that's miniflare)
