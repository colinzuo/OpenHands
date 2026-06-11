# Architecture — mental model

> Not a design doc. My current understanding of how the system fits together and how data flows. Update when I realize something was wrong or finally "get" a confusing part.

## The big picture

This repo is **not** the agent engine. It's the surrounding product:
- **Backend** (`openhands/`) — a FastAPI "app server" exposing the V1 REST API.
- **Frontend** (`frontend/`) — a React SPA (Vite, TanStack Query) talking to that API.
- **Enterprise** (`enterprise/`) — a separate Poetry project that dynamically extends the OSS server (auth, billing, integrations).

The actual agent / LLM / runtime logic comes from the **`openhands-agent-server`** pip dependency (pinned in `pyproject.toml`, sourced from the separate `software-agent-sdk` repo). Files like `openhands/sdk/...` or `openhands/workspace/...` are **not in this repo** — don't go looking for them here.

## Backend layering

```
openhands/server/listen.py        ← uvicorn entrypoint, ENABLE_V1 gate
        │ mounts
        ▼
openhands/app_server/app.py       ← builds FastAPI app: combines lifespans,
        │                            mounts MCP app, registers v1_router
        ▼
openhands/app_server/v1_router.py ← wires together per-domain routers
        │
        ▼
domain modules (each: *_router.py + service classes + storage models)
```

Domain modules under `app_server/`:
- `app_conversation/`, `sandbox/` — conversation lifecycle + execution environments
- `event/`, `event_callback/`, `pending_messages/` — event store/stream, webhooks, server-side message queue
- `user/`, `user_auth/`, `secrets/`, `settings/` — identity, auth, secrets, settings
- `git/`, `integrations/` — git provider integrations (github, gitlab, bitbucket, azure_devops, jira_dc, forgejo, ...)
- `mcp/`, `web_client/`, `config_api/`, `status/` — MCP proxy, SPA serving, config, health

## Frontend data flow (strict — do not violate)

```
UI components
   │  (never call src/api directly)
   ▼
TanStack Query hooks   src/hooks/query/use[Resource]   src/hooks/mutation/use[Action]
   ▼
Data Access Layer      src/api  (API client methods)
   ▼
Backend REST endpoints
```

Two settings-save patterns:
- **Immediate-save** — entity resources (API keys, secrets, MCP servers). Dedicated mutation hooks, no Save button.
- **Form-based** — LLM/app settings. Local `isDirty` tracking + `useSaveSettings`, explicit Save.

## Enterprise extension model

Separate Poetry project that extends the OSS server via **dynamic imports**. Key facts:
- Imports use **no `enterprise.` prefix** (`from storage.database import ...`) so the same code runs in both OSS and enterprise contexts.
- Adds Keycloak auth, Alembic migrations (`enterprise/migrations/`), Stripe billing, telemetry, richer integrations (incl. Slack/Linear).
- Has its own lint config; root ruff excludes `enterprise/`.

## Sandbox / credential flow (the subtle part)

SDK-created conversations inherit the user's SaaS credentials securely: raw secrets flow **SaaS → sandbox only**, never back through the SDK client (via `LookupSecret`). The SDK's `workspace.get_llm()` calls `GET /api/v1/users/me?expose_secrets=true`, which requires *both* a Bearer token and the `X-Session-API-Key` header. (Full details in root `AGENTS.md`.)

## Open questions / fuzzy areas
- _(none recorded yet — add as they come up, cross-reference investigations/)_
