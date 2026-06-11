# enterprise

`enterprise/` — separate Poetry project (Polyform Free Trial license, 30-day) that extends the OSS server via **dynamic imports**.

## What it adds on top of OSS
- **Auth**: Keycloak integration (vs OSS `default_user_auth`).
- **DB migrations**: Alembic, in `enterprise/migrations/versions/` (`alembic.ini`). CI checks for migration conflicts on PRs.
- **Billing**: Stripe (`integrations/stripe_service.py`).
- **Telemetry/analytics**: PostHog + a custom metrics framework.
- **Richer integrations** (`enterprise/integrations/`): GitHub, GitLab, Bitbucket(+DC), Azure DevOps, **Jira, Jira DC, Slack** — plus `manager.py`, `resolver_context.py`, `resolver_org_router.py`, multi-org routing. (OSS integrations are git-host only.)
- Entrypoints: `saas_server.py`, `run_maintenance_tasks.py`.

## Hard rules (will bite you)
- **Imports use no `enterprise.` prefix**: `from storage.database import a_session_maker`, NOT `from enterprise.storage.database import ...`. This keeps code runnable in both OSS and enterprise contexts. Same for `patch()` targets in tests (`telemetry.registry.logger`, not `enterprise.telemetry...`).
- Root ruff **excludes** `enterprise/`; it has its own `enterprise/dev_config/python/` lint config and pre-commit. Use `--show-diff-on-failure` to match CI.
- After bumping root `openhands-ai` version, run `poetry lock` inside `enterprise/`.

## Dev / test
```bash
cd enterprise && poetry install --with dev,test          # slow, be patient
poetry run pre-commit install --config ./dev_config/python/.pre-commit-config.yaml
make start-backend   # or: make run
# tests (set PYTHONPATH):
PYTHONPATH=".:$PYTHONPATH" poetry run pytest tests/unit/<module>/ --confcutdir=tests/unit/<module>
```
- Unit tests use **in-memory SQLite** (`sqlite:///:memory:`) + module-local `conftest.py`; mock external deps (`AsyncMock`/`MagicMock`). Real DB only for integration tests.
- Build the main project first (`make build`) so the OSS package + frontend exist.

## Notes
- Recent org work (single-org installs, default-org owner, `hide_personal_workspaces`) lives here — see recent git log. Org selection on the frontend uses `useOrganizations()` + selected-org store.
