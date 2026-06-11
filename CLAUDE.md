# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> A much longer `AGENTS.md` lives at the repo root with detailed implementation notes (conversation state, microagents, adding LLM models/settings, sandbox credential flow). Read it when working on those specific areas. This file is the quick-start.

## What this repo is

This is the **OpenHands GUI / Cloud / Enterprise** layer — the REST API, React frontend, and hosted deployment. The agentic engine itself lives in a separate repo (`software-agent-sdk`) and is consumed here as the `openhands-agent-server` pip dependency (pinned in `pyproject.toml`). When a task references SDK internals (`openhands/sdk/...`, `openhands/workspace/...`), those files are **not** in this repo.

- Python backend in `openhands/` (the V1 application server) — Python 3.12/3.13, Poetry.
- React frontend in `frontend/` — Node 22.x, npm, Vite, vitest, TanStack Query.
- Source-available enterprise layer in `enterprise/` (separate Poetry project, Polyform license).

## Critical workflow rules

- **ALWAYS run `make install-pre-commit-hooks` before making changes.** Pre-commit hooks MUST pass before pushing.
- Backend lint: `pre-commit run --config ./dev_config/python/.pre-commit-config.yaml` (runs on staged files; fix mypy/ruff/whitespace issues, re-run until green).
- Frontend lint/build: `cd frontend && npm run lint:fix && npm run build`
- If GitHub CI fails but local lint passes, add `--show-diff-on-failure` to match CI exactly.
- Prefer `git add <file>` over `git add .` to avoid staging stray files.
- PR-only artifacts (design notes, debug logs) go in a `.pr/` directory at repo root — auto-cleaned on approval, never merged.

## Common commands

### Setup / run
```bash
make build                  # full setup: deps (backend+frontend) + hooks + frontend build
# Run full app locally (no Docker, local runtime):
export INSTALL_DOCKER=0 RUNTIME=local
make run FRONTEND_PORT=12000 FRONTEND_HOST=0.0.0.0 BACKEND_HOST=0.0.0.0
make start-backend          # backend only (uvicorn openhands.server.listen:app, hot reload)
make start-frontend         # frontend only
```
The V1 app server is included in `openhands.server.listen:app` by default unless `ENABLE_V1=0`.

### Backend tests (pytest)
```bash
poetry run pytest tests/unit/test_xxx.py          # one file
poetry run pytest tests/unit/test_xxx.py::test_fn # one test
```
All tests are under `tests/unit/` (mirrors source layout: `app_server/`, `server/`, `integrations/`, `mcp/`, `storage/`, ...). Write tests with pytest; `asyncio_mode = auto` is set.

### Frontend (run inside `frontend/`)
```bash
npm run test                  # vitest
npm run test -- -t "TestName" # single test by name
npm run build
npm run lint:fix
npm run make-i18n             # regenerate i18n declaration file
```

### Lockfiles
When regenerating `poetry.lock` / `uv.lock`, use the **same tool version** that generated the file (read the version from the lockfile header) to avoid diff noise. See AGENTS.md for the exact commands.

## Architecture

### Backend (`openhands/`)
- `openhands/app_server/` — the V1 FastAPI application. Organized by domain module, each typically with a `*_router.py` (endpoints), service classes, and storage models:
  - `app_conversation/`, `sandbox/` — sandboxed conversation lifecycle and execution environments
  - `event/`, `event_callback/`, `pending_messages/` — event storage/streaming, webhooks, server-side message queuing
  - `user/`, `user_auth/`, `secrets/`, `settings/` — identity, auth, secrets, user/app settings
  - `git/`, `integrations/` — git provider integrations (github, gitlab, bitbucket, azure_devops, jira_dc, forgejo, ...)
  - `mcp/`, `web_client/`, `config_api/`, `status/` — MCP proxy, SPA serving, config + health endpoints
  - `app.py` composes the FastAPI app (combines lifespans, mounts MCP app, registers `v1_router`).
- `openhands/server/` — the outer server entrypoint (`listen.py`) that mounts the V1 routes.
- The actual agent/LLM/runtime logic comes from the `openhands-agent-server` SDK dependency, not from this repo.

### Frontend (`frontend/src/`)
Strict data-flow layering — **do not violate**:
```
UI components → TanStack Query hooks → Data Access Layer (src/api) → API endpoints
```
- `src/api` API client methods are **never** called directly from components — always wrap in TanStack Query.
- Query hooks in `src/hooks/query/`, named `use[Resource]` (e.g. `useConversationSkills`).
- Mutation hooks in `src/hooks/mutation/`, named `use[Action]` (e.g. `useDeleteConversation`).
- Two settings-save patterns: **immediate-save** (entity resources like API keys, secrets, MCP servers — dedicated mutation hooks, no Save button) vs **form-based** (LLM/app settings — `isDirty` tracking + `useSaveSettings`). See AGENTS.md.

### Enterprise (`enterprise/`)
Separate Poetry project that extends the OSS server via dynamic imports. Adds Keycloak auth, Alembic migrations (`enterprise/migrations/`), Stripe billing, telemetry, and richer integrations (GitHub/GitLab/Jira/Linear/Slack).
- Use relative imports **without** the `enterprise.` prefix (e.g. `from storage.database import ...`) so code works in both contexts.
- Tests use in-memory SQLite (`sqlite:///:memory:`) and module-local `conftest.py`; set `PYTHONPATH=".:$PYTHONPATH"`.
- Enterprise lint uses its own config: `poetry run pre-commit run --all-files --show-diff-on-failure --config ./dev_config/python/.pre-commit-config.yaml` (root ruff excludes `enterprise/`).
- After bumping the root `openhands-ai` version, run `poetry lock` in `enterprise/`.

### VS Code extension (`openhands/app_server/integrations/vscode/`)
Node project. `npm run lint:fix`, `npm run typecheck`, `npm run compile`, `npm run package-vsix`. Use `vscode.window.createOutputChannel()` for debug logging, not error popups.

## Code style
- Ruff with **single quotes** for inline strings, double quotes for docstrings (config in `pyproject.toml`; `E501` line-length is ignored). `enterprise/` is excluded from root ruff.
- Env-var boolean toggles **must** accept both `'true'` and `'1'`: `os.getenv('X', 'false').lower() in ('true', '1')` — older Helm charts default to `'1'`. Add a unit test for the `'1'` case.

## Microagents
Repo-specific microagents live in `.openhands/microagents/` (Markdown + optional frontmatter triggers). Files without frontmatter are always loaded; files with `triggers:` load only on keyword match.

## PRs
Follow `.github/pull_request_template.md` — it has a `HUMAN:` section, a human-tested checkbox, and an `AGENT:` section. Pin third-party GitHub Actions to a 40-char commit SHA with a version tag in a trailing comment.
