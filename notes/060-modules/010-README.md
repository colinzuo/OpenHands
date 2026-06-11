# Module notes

Per-module mental models for the backend (`openhands/app_server/`) and the frontend/enterprise layers. One file per major module. Promote durable cross-module facts up to [../020-architecture.md](../020-architecture.md).

## The shared backend pattern (read this first)

Almost every `app_server` module is built the same way — recognize it once and every module reads the same:

```
<module>_models.py     ← Pydantic models (request/response/storage)
<module>_router.py     ← FastAPI endpoints
<module>_service.py    ← abstract base service (ABC) + an Injector subclass
<backend>_<module>_service.py
                       ← concrete backends: filesystem / sql / docker / remote /
                         aws / google_cloud / process ...
```

- **Abstract service**: defines the CRUD/operations contract (e.g. `EventService`, `SandboxService`).
- **Pluggable backends**: multiple concrete impls; which one runs is chosen by config, not hardcoded.
- **`Injector`** (`services/injector.py`): generic DI helper. `.depends()` plugs into FastAPI `Depends(...)`; `.inject()/.context()` share state across nested injectors via Starlette request `state`.
- **Wiring**: `app_server/config.py` exposes `get_<module>_service(...)` factories (`get_event_service`, `get_sandbox_service`, `get_app_conversation_service`, ...) returning async context managers. This is the single place backend selection happens (OSS vs SaaS/enterprise variants too).

When adding a feature: add to the abstract service contract first, then each backend, then expose via the router. When debugging "which implementation is actually running?", look at `config.py`.

## Index (ordered by dependency: foundational → high-level)
- [020-user-and-auth.md](020-user-and-auth.md) — user context, user_auth, JWT service (foundational).
- [030-settings-and-secrets.md](030-settings-and-secrets.md) — user/app settings, LLM profiles, secrets.
- [040-sandbox.md](040-sandbox.md) — execution environments (Docker/remote/process).
- [050-event.md](050-event.md) — event store/stream, callbacks/webhooks, pending messages.
- [060-integrations-and-git.md](060-integrations-and-git.md) — git providers + provider dispatch.
- [070-mcp.md](070-mcp.md) — MCP server mount + Tavily proxy (uses integrations).
- [080-app-conversation.md](080-app-conversation.md) — orchestrates sandbox + event + git.
- [090-frontend.md](090-frontend.md) — React SPA layering (depends on backend API).
- [100-enterprise.md](100-enterprise.md) — enterprise extension layer (extends everything).
