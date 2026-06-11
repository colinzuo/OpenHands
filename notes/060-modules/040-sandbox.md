# sandbox

`openhands/app_server/sandbox/`

## What it does
Manages the **execution environments** agents run inside (containers/remote machines/local processes) so agent actions can't harm the host. Lifecycle: create, start, stop, destroy. User-scoped access control.

## Key files
- `sandbox_models.py`, `sandbox_spec_models.py` — runtime + spec (template) models.
- `sandbox_router.py`, `sandbox_spec_router.py` — endpoints.
- `sandbox_service.py`, `sandbox_spec_service.py` — abstract contracts.
- Backends: `docker_sandbox_service.py`, `remote_sandbox_service.py`, `process_sandbox_service.py` (+ matching `*_spec_service.py`, and `preset_sandbox_spec_service.py`).
- `session_auth.py` — validates the `X-Session-API-Key` that identifies an active sandbox.

## Two concepts
- **Sandbox** = a running instance.
- **Sandbox spec** = the template/definition a sandbox is created from.

## Backend selection
`RUNTIME=local` (+ `INSTALL_DOCKER=0`) → process backend for local dev; Docker/remote in hosted setups. Chosen via `config.get_sandbox_service(...)`.

## Notes / gotchas
- `session_auth.py` is central to the **credential inheritance** flow: the `X-Session-API-Key` header proves a caller owns an active sandbox, gating `expose_secrets=true` on `/api/v1/users/me` (see [../020-architecture.md](../020-architecture.md) and root AGENTS.md). Raw secrets flow SaaS → sandbox only.
- A conversation with `sandbox_status == "MISSING"` is **archived** — frontend renders read-only (see AGENTS.md → `useAgentState`).
- Local-runtime startup quirks (stale tmux session, Playwright browsers) are documented in AGENTS.md troubleshooting.
