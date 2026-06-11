# user / user_auth / services (JWT)

`openhands/app_server/user/`, `user_auth/`, `services/`

## user/ — user context + endpoints
- `user_context.py` — abstract `UserContext` for user-scoped operations.
- `auth_user_context.py` — compatibility layer bridging `user_auth`.
- `specifiy_user_context.py` — explicit/specified user context (note the original spelling in code).
- `user_router.py` — user endpoints, incl. `GET /api/v1/users/me` (and `?expose_secrets=true`).
- `skills_router.py` — user skills endpoints.
- `UserContextInjector` — FastAPI DI factory to resolve the current user's context.

## user_auth/ — pluggable auth
- `user_auth.py` — abstract auth contract.
- `default_user_auth.py` — OSS default implementation. (SaaS/enterprise swap in their own via dynamic imports.)

## services/ — JWT
- `services/jwt_service.py` — `JwtService`: signing, verification, **JWE encryption** for sensitive payloads, multi-key + rotation, configurable algs (RS256/HS256).
- Also `db_session*.py`, `httpx_client_injector.py`, `injector.py` (the generic DI base — see [README](010-README.md)).

## Auth flow for `expose_secrets=true` (important)
`GET /api/v1/users/me?expose_secrets=true` returns unmasked secrets and requires **both**:
1. Bearer token (`OPENHANDS_API_KEY`) — proves user identity.
2. `X-Session-API-Key` header — proves caller owns an active sandbox (validated in [sandbox](040-sandbox.md) `session_auth.py`).

Called by the SDK's `workspace.get_llm()`. Without `expose_secrets`, secrets are masked.

## Notes
- In this sandbox env, an inherited `SESSION_API_KEY` can cause `/api/v1/settings` 401s in the browser — unset before `make run` (AGENTS.md).
