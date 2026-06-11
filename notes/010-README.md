# Notes index

Entry point when coming back after a break. Keep short and current.

## Files
- [015-usage.md](015-usage.md) — how this notes system works (the guide).
- [020-architecture.md](020-architecture.md) — evolving mental model of the system (architecture + data flow).
- [030-decisions.md](030-decisions.md) — non-trivial choices and their rationale. Append-only.
- [040-TODO.md](040-TODO.md) — actionable tasks + blockers.
- [050-investigations/](050-investigations/) — one file per question/bug/idea, named `YYYY-MM-DD-topic.md`.
- [060-modules/](060-modules/) — per-module mental models (backend modules, frontend, enterprise). Start with its README for the shared service/injector pattern.

## What this repo is (one-liner)
OpenHands **GUI / Cloud / Enterprise** layer: REST API (`openhands/`), React frontend (`frontend/`), enterprise extensions (`enterprise/`). The agent engine itself lives in the separate `software-agent-sdk` repo, consumed here as the `openhands-agent-server` pip dependency.

## Where to start in code
- Backend app: `openhands/app_server/app.py` → mounts `v1_router`.
- Server entrypoint: `openhands/server/listen.py` (`uvicorn openhands.server.listen:app`).
- Frontend data flow: `UI → src/hooks/{query,mutation} → src/api`.

## See also
- `CLAUDE.md` (repo root) — quick-start for agents.
- `AGENTS.md` (repo root) — long-form implementation notes.
