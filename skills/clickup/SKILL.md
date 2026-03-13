---
name: clickup
description: Use this skill whenever the user asks to inspect, search, create, update, comment on, or organize ClickUp tasks or docs, especially when they mention a ClickUp URL, task ID, list ID, doc ID, page ID, assignee, status, due date, checklist, comment, or document workflow. 处理 ClickUp 任务、评论、文档、页面、状态、指派、截止时间、checklist 等请求时也应触发。 This skill uses the globally installed `clickup` CLI instead of repo-local scripts.
---

# ClickUp

Use the globally installed `clickup` CLI to work with ClickUp tasks, comments, and docs.

Open `README.md` in this directory when you need installation steps, environment-variable setup, or the full command reference.

## Operating rules

- Never read `.env` files or print secrets.
- Assume configuration comes from shell environment variables, not from files in the repo.
- If `clickup` is unavailable or auth is not configured, stop and point the user to `README.md`.
- Prefer `--json` when the result will be parsed, transformed, or used in follow-up automation.
- Prefer reading current state before mutating it, unless the user's requested change is already precise and unambiguous.

## First checks

1. Confirm the CLI exists with `clickup --help`.
2. If the command fails because the CLI is missing, tell the user to install it using the instructions in `README.md`.
3. If the API returns an auth or workspace error, tell the user to configure the required shell environment variables from `README.md`.

## Command selection

| User intent | Command |
| --- | --- |
| Read a task from a ClickUp URL or task ID | `clickup get <url-or-id>` |
| Read task comments | `clickup comments <url-or-id>` |
| Add a comment | `clickup comment <url-or-id> "message"` |
| List or update status | `clickup status <url-or-id> [status]` |
| List tasks in a list | `clickup tasks <list_id>` |
| Show current user | `clickup me` |
| Create a task | `clickup create [list_id] "title"` |
| List tasks assigned to the current user | `clickup my-tasks` |
| Search tasks | `clickup search "query"` |
| Assign a task | `clickup assign <task> <user>` |
| Set due date | `clickup due <task> "date"` |
| Set priority | `clickup priority <task> <level>` |
| Create a subtask | `clickup subtask <task> "title"` |
| Move a task to another list | `clickup move <task> <list_id>` |
| Add an external link note | `clickup link <task> <url> ["description"]` |
| Add a checklist item | `clickup checklist <task> "item"` |
| Delete a comment | `clickup delete-comment <comment_id>` |
| Notify a watcher via mention comment | `clickup watch <task> <user>` |
| Add a tag | `clickup tag <task> "tag_name"` |
| Replace task description with Markdown | `clickup description <task> "text"` |
| List or search docs | `clickup docs ["query"]` |
| Read doc details | `clickup doc <doc_id>` |
| Create a doc | `clickup create-doc "title" [--content "..."]` |
| Read a doc page | `clickup page <doc_id> <page_id>` |
| Create a doc page | `clickup create-page <doc_id> "title" [--content "..."]` |
| Update a doc page | `clickup edit-page <doc_id> <page_id> [--name "..."] [--content "..."]` |

## Recommended workflow

1. Normalize the target the user gave you: task URL, task ID, list ID, doc ID, or page ID.
2. Run the narrowest read command first when you need context: `get`, `comments`, `tasks`, `docs`, `doc`, or `page`.
3. For write operations, choose the smallest command that matches the requested change.
4. Re-read the updated object only when the user needs confirmation or the write response is not sufficient.
5. Summarize only the fields that matter to the user instead of dumping raw output unless they explicitly asked for it.

## Practical patterns

- Use `clickup get <task> --json` when you need stable field names.
- Use `clickup status <task>` first when you need to inspect valid statuses before updating one.
- Status updates are case-insensitive and allow partial matching.
- Use `clickup tasks <list_id> --me` to filter a list to the current user.
- `assign` and `watch` can resolve users by username, email, or user ID.
- Use `clickup create "Quick task"` only when `CLICKUP_DEFAULT_LIST_ID` is configured.
- `clickup due <task> clear` or `clickup due <task> none` clears a due date; explicit dates like `2026-03-20` are also valid.
- Use `clickup docs "keyword"` before creating a new doc if duplicates are possible.
- `clickup edit-page <doc_id> <page_id>` requires at least one of `--name` or `--content`.
- Use `clickup edit-page <doc_id> <page_id> --name "..." --content "..."` when renaming and updating content together.

## Troubleshooting

- `clickup: command not found`: install the global CLI first.
- Auth failures: `CLICKUP_API_TOKEN` is missing or invalid.
- Workspace/list errors: set `CLICKUP_WORKSPACE_ID` or `CLICKUP_DEFAULT_LIST_ID` as needed.
- `watch` does not add a native ClickUp watcher; it posts an `@mention` comment because the API does not expose direct watcher management.
