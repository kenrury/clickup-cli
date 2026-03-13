# Repository Guidelines

## Project Structure & Module Organization
This repository is a globally installable Node CLI package plus a bundled Codex skill reference. The package manifest lives at `package.json`. Runtime code is organized as follows:

- `bin/clickup.js`: global CLI entrypoint.
- `src/cli/`: argument parsing, command registration, help output, and handlers.
- `src/services/`: ClickUp task, comment, doc, and user workflows.
- `src/http/`: shared ClickUp HTTP client.
- `src/utils/`: ID parsing, date parsing, Markdown conversion, and formatters.
- `tests/`: automated coverage with `node:test`.
- `skills/clickup/`: skill-facing documentation for agents that should call the global `clickup` binary.

Keep runtime code under `src/` and keep skill guidance under `skills/clickup/` instead of reintroducing repo-local script entrypoints.

## Build, Test, and Development Commands
There is no build step; the CLI runs directly with Node.

- `pnpm install`: install package metadata and any future dependencies.
- `node bin/clickup.js --help`: verify the local CLI entrypoint and command registration.
- `node bin/clickup.js me`: verify local auth and current user detection.
- `node bin/clickup.js get <task-id>`: exercise the main task lookup path.
- `pnpm test`: run the automated test suite.
- `npm run pack:check`: inspect the npm publish payload before release.

Prefer running commands against test tasks or docs when changing API behavior.

## Coding Style & Naming Conventions
Use ES modules (`.mjs`), 2-space indentation, semicolons, and single quotes, matching the files under `src/`. Keep command handlers explicit and side effects near the CLI layer. Use descriptive verb-based function names such as `getTask`, `createPage`, or `formatTaskList`. Put network calls in `src/services/` and `src/http/`, and keep pure helpers in `src/utils/`.

## Testing Guidelines
Automated tests are configured with `node:test`. Validate changes with `pnpm test` plus focused CLI invocations and `--json` output where helpful. Keep new tests under `tests/` with a clear `*.test.mjs` name.

## Commit & Pull Request Guidelines
This checkout does not include Git history, so no local commit convention can be inferred. Use short imperative subjects with a clear scope, for example `clickup: add doc page rename support`. In pull requests, include:

- a brief problem statement and approach
- sample commands used for verification
- notes about new env vars, API fields, or breaking command changes

## Security & Configuration Tips
Never read, print, or commit `.env` files or any secrets. The CLI reads configuration from shell environment variables such as `CLICKUP_API_TOKEN`, `CLICKUP_WORKSPACE_ID`, `CLICKUP_USER_ID`, and `CLICKUP_DEFAULT_LIST_ID`; keep setup guidance in `README.md` and `skills/clickup/README.md`. Redact task URLs, IDs, and tokens in logs or screenshots shared in reviews.
