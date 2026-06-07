# AGENTS.md

## Overview

GKD subscription template — TypeScript project that builds an Android subscription file (`dist/gkd.json5`) consumed by the GKD app.

- **Package manager**: pnpm >= 9 (locked at 10.31.0)
- **Node**: >= 22 (required by WasmGc for Java/Kotlin regex validation)
- **Runtime for scripts**: `tsx` (scripts run as TypeScript, not pre-compiled)

## Commands

```sh
pnpm install          # install deps + init git hooks
pnpm run check        # tsc --noEmit + subscription validation (scripts/check.ts)
pnpm run build        # tsc --noEmit + build dist output (scripts/build.ts)
pnpm run format       # prettier
pnpm run lint         # eslint --fix
```

Always run `pnpm install` after `package.json` changes.

## Build dependency

`scripts/build.ts` imports `subscription` from `./check`, not from `../src/subscription` directly. This means `pnpm run check` must pass for `pnpm run build` to succeed — `check` is a prerequisite at runtime, not just a CI step.

## Source structure

```
src/
  subscription.ts   # subscription metadata (id, name, author) — **must set a unique id**
  categories.ts     # rule categories
  globalGroups.ts   # global rules
  apps/             # per-app rule files (one file per Android package, e.g. com.tencent.mm.ts)
scripts/
  check.ts          # validates subscription + API version
  build.ts          # generates dist/ output
dist/               # built output (gkd.json5, gkd.version.json5, CHANGELOG.md)
```

Per-app files in `src/apps/` are named by Android package ID (e.g. `com.tencent.mm.ts`) and auto-imported via `batchImportApps`.

## Code style

- **Single quotes** required (no template literals unless expression needed) — enforced by ESLint
- **Trailing commas** always — enforced by Prettier
- **2-space indentation** — enforced by Prettier
- **Unused imports will error** — `eslint-plugin-unused-imports`

## Git hooks (auto-installed on `pnpm install`)

- **pre-commit**: runs `lint-staged` (eslint + prettier on staged files)
- **pre-push**: runs `pnpm run check`

## CI workflows

| Workflow | Trigger | What it does |
|---|---|---|
| `pull_request_check.yml` | PR to `main` | check + format + lint; **enforces <= 1 source file changed per PR** |
| `check_fix_push.yml` | Push to any branch | check + format + lint, auto-commits fixes as `chore(actions): check_format_lint` |
| `build_release.yml` | Manual (`workflow_dispatch`) | build + tag + create GitHub Release |

## Custom build config

Optional `gkd` key in `package.json` overrides build defaults:

```json
{
  "gkd": {
    "outDir": "dist",
    "file": "gkd.json5",
    "versionFile": "gkd.version.json5",
    "changelog": "CHANGELOG.md"
  }
}
```

## Key constraint

The `id` field in `src/subscription.ts` must be unique across all GKD subscriptions to avoid conflicts. Search existing IDs before choosing one.
