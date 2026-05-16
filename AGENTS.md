# OpenCode Repo Notes

- Default branch is `dev`; `origin/HEAD` points to `origin/dev`. Diff and open PRs against `dev` or `origin/dev`, not `main`.
- Use Bun `1.3.11` from the root `packageManager`. `.husky/pre-push` checks the Bun version and runs `bun typecheck`.
- Root commands: `bun dev` runs the CLI/TUI from `packages/opencode`; `bun typecheck` is the repo-wide Turbo check; `bun lint` runs `oxlint`; root `bun test` is an intentional guard that always fails.
- Run tests from package directories or with `--cwd`, not from repo root.

## Package Map

<<<<<<< HEAD
- `packages/opencode`: main CLI, headless server, and TUI. Entry point is `packages/opencode/src/index.ts`.
- `packages/app`: shared web UI used by the desktop shells and Playwright e2e tests.
- `packages/desktop`: Tauri desktop wrapper around `packages/app`.
- `packages/desktop-electron`: Electron desktop wrapper around `packages/app`.
- `packages/console/app`: separate console/site app, not the same app as `packages/app`.
=======
- Keep things in one function unless composable or reusable
- Do not extract single-use helpers preemptively. Inline the logic at the call site unless the helper is reused, hides a genuinely complex boundary, or has a clear independent name that improves the caller.
- Avoid `try`/`catch` where possible
- Avoid using the `any` type
- Use Bun APIs when possible, like `Bun.file()`
- Rely on type inference when possible; avoid explicit type annotations or interfaces unless necessary for exports or clarity
- Prefer functional array methods (flatMap, filter, map) over for loops; use type guards on filter to maintain type inference downstream
- In `src/config`, follow the existing self-export pattern at the top of the file (for example `export * as ConfigAgent from "./agent"`) when adding a new config module.
>>>>>>> 77e6c0d329ee568818bcf495f2b8b858b286e453

## Generated Code

- If you change API surfaces or SDK-related files, run `./script/generate.ts` from the repo root. It rebuilds the JS SDK, refreshes OpenAPI output from `packages/opencode`, then runs repo formatting.
- To rebuild only the JavaScript SDK, run `./packages/sdk/js/script/build.ts`.
- `packages/opencode/script/build.ts` is the CLI release build. It builds `packages/app` and embeds that web UI into the binary unless `--skip-embed-web-ui` is passed.

## Verification

- CLI/server package: `bun --cwd packages/opencode test` or `bun --cwd packages/opencode test:ci`.
- Web app unit tests: `bun --cwd packages/app test:unit`.
- Web app e2e: `bun --cwd packages/app test:e2e:local -- --grep "<name>"` for a focused run.
- Playwright starts the Vite app itself, but it expects an OpenCode backend at `127.0.0.1:4096` unless `PLAYWRIGHT_SERVER_HOST` / `PLAYWRIGHT_SERVER_PORT` are overridden.

## Frontend Gotcha

- Do not use `opencode dev web` or root `bun dev web` to verify local UI/CSS changes. When embedded UI is disabled, the server route proxies `https://app.opencode.ai`.
- For local UI work, run the backend and app separately: from `packages/opencode`, `bun run --conditions=browser ./src/index.ts serve --port 4096`; from `packages/app`, `bun dev -- --port 4444`; then open `http://localhost:4444`.

## PR Rules

- PR titles are enforced by CI: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, or `test:` with optional scope.
- Non-`docs`/`refactor`/`feat` PRs must link an issue with `Fixes #...` or `Closes #...`.
- PR bodies are checked for the standard template sections from `.github/pull_request_template.md`.

## Local Instructions

<<<<<<< HEAD
- Before editing inside `packages/opencode`, `packages/opencode/test`, `packages/app`, `packages/desktop`, or `packages/desktop-electron`, read that directory's own `AGENTS.md` first.
- Prefer executable config over package READMEs when they conflict. `packages/app/README.md` and `packages/opencode/README.md` contain stale template-era instructions.
=======
```ts
// Good
const foo = condition ? 1 : 2

// Bad
let foo
if (condition) foo = 1
else foo = 2
```

### Control Flow

Avoid `else` statements. Prefer early returns.

```ts
// Good
function foo() {
  if (condition) return 1
  return 2
}

// Bad
function foo() {
  if (condition) return 1
  else return 2
}
```

### Complex Logic

When a function has several validation branches or supporting details, make the main function read as the happy path and move supporting details into small helpers below it.

```ts
// Good
export function loadThing(input: unknown) {
  const config = requireConfig(input)
  const metadata = readMetadata(input)
  return createThing({ config, metadata })
}

function requireConfig(input: unknown) {
  ...
}
```

- Keep helpers close to the code they support, below the main export when that improves readability.
- Do not over-abstract simple expressions into many single-use helpers; extract only when it names a real concept like `requireConfig` or `readMetadata`.
- Do not return `Effect` from helpers unless they actually perform effectful work. Synchronous parsing, validation, and option building should stay synchronous.
- Prefer Effect schema helpers such as `Schema.UnknownFromJsonString` and `Schema.decodeUnknownOption` over manual `JSON.parse` wrapped in `Effect.try` when parsing untrusted JSON strings.
- Add comments for non-obvious constraints and surprising behavior, not for obvious assignments or control flow.

### Schema Definitions (Drizzle)

Use snake_case for field names so column names don't need to be redefined as strings.

```ts
// Good
const table = sqliteTable("session", {
  id: text().primaryKey(),
  project_id: text().notNull(),
  created_at: integer().notNull(),
})

// Bad
const table = sqliteTable("session", {
  id: text("id").primaryKey(),
  projectID: text("project_id").notNull(),
  createdAt: integer("created_at").notNull(),
})
```

## Testing

- Avoid mocks as much as possible
- Test actual implementation, do not duplicate logic into tests
- Tests cannot run from repo root (guard: `do-not-run-tests-from-root`); run from package dirs like `packages/opencode`.

## Type Checking

- Always run `bun typecheck` from package directories (e.g., `packages/opencode`), never `tsc` directly.
>>>>>>> be6a89a3b89bcdbd359948078360591a84e91f04
