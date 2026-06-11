# app_conversation

`openhands/app_server/app_conversation/`

## What it does
Manages **app conversations** — the OpenHands-side record + lifecycle of a conversation that runs inside a sandbox. Creation, retrieval, status tracking, search/filter, pagination.

## Key files
- `app_conversation_models.py` — Pydantic models.
- `app_conversation_router.py` — FastAPI endpoints.
- `app_conversation_service_base.py` / `app_conversation_service.py` — abstract contract.
- `live_status_app_conversation_service.py` — real-time status (the "live" view; reads execution status, not just stored state).
- `sql_app_conversation_info_service.py` / `app_conversation_info_service.py` — stored conversation metadata.
- `app_conversation_start_task_service.py` + `sql_...` — the start-task workflow (kicking off a conversation in a sandbox).
- `hook_loader.py`, `skill_loader.py`, `git/` — load hooks/skills and git context into a conversation.

## Data flow
1. Client requests a new conversation → router → `AppConversationService.create`.
2. Service provisions/attaches a **sandbox** (see [sandbox.md](040-sandbox.md)) and records a start task.
3. `LiveStatusAppConversationService` overlays real-time status on stored info.
4. Events for the conversation flow through the [event](050-event.md) module.

## Notes / gotchas
- Two service axes: **info** (stored metadata, SQL-backed) vs **live status** (runtime). Don't confuse them when debugging stale state.
- This is the orchestration hub — it ties sandbox + events + git + skills together. The actual agent loop lives in the `openhands-agent-server` SDK dependency, not here.
