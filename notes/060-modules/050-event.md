# event / event_callback / pending_messages

`openhands/app_server/event/`, `event_callback/`, `pending_messages/`

Grouped because they form the conversation event pipeline.

## event/ — storage, retrieval, streaming
- `event_service_base.py` / `event_service.py` — abstract contract + injector.
- `event_store.py` — storage abstraction.
- Backends: `filesystem_event_service.py`, `aws_event_service.py`, `google_cloud_event_service.py` (and DB-backed per README).
- `event_router.py` — query by conversation ID, filter by kind/timestamp, sort, paginate, plus **real-time streaming**.
- Wired via `config.get_event_service(...)`.

## event_callback/ — webhooks
- `event_callback_service.py` + `sql_event_callback_service.py` — register/manage callbacks (SQL-backed).
- `webhook_router.py` — webhook endpoints; retry logic + auth for delivery.
- `set_title_callback_processor.py` — concrete processor example (auto-sets conversation title from events).
- `event_callback_result_models.py` — tracks delivery results/status.

## pending_messages/ — server-side queue
- `pending_message_service.py` + `pending_message_router.py` — queues messages server-side (e.g. messages sent while the agent/sandbox isn't ready to consume them yet).

## Data flow
agent/sandbox produces events → `EventService` persists + streams → registered `event_callback`s fire (webhooks, title-setter) → clients read via `event_router` (poll or stream). `pending_messages` buffers inbound messages until consumable.

## Notes
- Storage backend varies by deployment (filesystem locally, cloud object stores hosted) — check `config.py` to know which is live.
