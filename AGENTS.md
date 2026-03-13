# Repository Guidelines

## Project Structure & Module Organization
This repository is a small Node-based Codex skill rather than a full application. The package manifest lives at `package.json`. The implementation is under `.agents/skills/clickup/`:

- `query.mjs`: CLI entrypoint for task and document commands.
- `api/`: ClickUp API clients grouped by domain (`tasks.mjs`, `docs.mjs`, `comments.mjs`).
- `lib/`: shared parsing and formatting helpers.
- `SKILL.md`: end-user usage and setup notes.
- `.env-example`: template for local configuration. Never commit a real `.env`.

Keep new code in the existing `api/` or `lib/` folders instead of adding ad hoc top-level files.

## Build, Test, and Development Commands
There is no build step; the CLI runs directly with Node.

- `pnpm install`: install package metadata and any future dependencies.
- `node .agents/skills/clickup/query.mjs me`: verify local auth and cached user detection.
- `node .agents/skills/clickup/query.mjs get <task-id>`: exercise the main task lookup path.
- `pnpm test`: currently a placeholder that exits with an error; replace it when adding real tests.

Prefer running commands against test tasks or docs when changing API behavior.

## Coding Style & Naming Conventions
Use ES modules (`.mjs`), 2-space indentation, semicolons, and single quotes, matching `query.mjs`. Keep command handlers explicit and side effects near the CLI layer. Use descriptive verb-based function names such as `getTask`, `createPage`, or `formatTaskList`. Put network calls in `api/` and pure helpers in `lib/`.

## Testing Guidelines
Automated tests are not configured yet. For now, validate changes with focused CLI invocations and `--json` output where possible. When adding tests, place them under `tests/` or beside the module with a clear `*.test.mjs` name, and wire them into `pnpm test`.

## Commit & Pull Request Guidelines
This checkout does not include Git history, so no local commit convention can be inferred. Use short imperative subjects with a clear scope, for example `clickup: add doc page rename support`. In pull requests, include:

- a brief problem statement and approach
- sample commands used for verification
- notes about new env vars, API fields, or breaking command changes

## Security & Configuration Tips
Copy `.agents/skills/clickup/.env-example` to a local `.env`, but never read, print, or commit secrets. Redact task URLs, IDs, and tokens in logs or screenshots shared in reviews.
