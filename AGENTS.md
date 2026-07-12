# Repository Guidelines

## Project Structure & Module Organization

This repository currently tracks the design and operating state of PoliBot, an
ESPOL academic assistant orchestrated in n8n. It does not yet contain exported
n8n workflows, application source code, or automated tests.

- `docs/product/espol-academic-agent.md` is the product specification.
- `docs/architecture/` records workflow design and integration contracts.
- `docs/operations/implementation-status.md` records the implemented MVP,
  workflow IDs, and verified behavior. Update it when workflow behavior changes.
- `docs/plans/` contains the staged delivery plan and exit criteria.
- `README.md` is the short project entry point.
- `.opencode/memory/` is local working memory; keep it untracked and do not
  treat it as product documentation.

Keep any future workflow exports in a clearly named directory such as
`workflows/`, using names that match the n8n workflow (for example,
`wf_telegram_router.json`). Keep reusable scripts and tests under `scripts/`
and `tests/` respectively.

## Development & Validation

There are no repository-managed build, lint, or test commands at present. The
MVP is validated in the running n8n instance: trigger the Telegram flow, check
the Canvas response and fallback path, then inspect the execution through MCP.
Do not claim a workflow is tested solely from an isolated node run; the Canvas
input context is required. When adding a runtime, document its exact commands
in `README.md` and add a repeatable test command.

## Coding Style & Naming Conventions

Write project documentation in Spanish, matching the existing specifications.
Use descriptive headings, short sections, and Markdown code fences for payloads
or commands. Name n8n workflows with lowercase snake case and an `wf_` prefix,
such as `wf_cmd_canvas_queries`. Keep bot commands and user-facing replies in
Spanish. Format Ecuador-facing dates with `America/Guayaquil`; Canvas todo items
may provide deadlines through `assignment.due_at` rather than `due_at`.

## Security & Configuration

Never commit `.env`, Canvas personal access tokens, Telegram credentials, OAuth
refresh tokens, or execution payloads containing student data. Use n8n
credentials and environment variables instead. Provide sanitized examples in
`.env.example` if configuration is added.

## Commits & Pull Requests

Follow the existing Conventional Commit style: `feat: add docs and memory state`.
Use concise imperative summaries, for example `fix: handle Canvas todo due dates`.
Pull requests should explain the user-visible workflow change, list affected
integrations, link the relevant issue when available, and include sanitized
execution evidence or screenshots for Telegram/n8n changes.
