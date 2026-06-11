# settings / secrets

`openhands/app_server/settings/`, `secrets/`

## settings/
- `settings_models.py` — `Settings` model (the big one), plus `GETSettingsModel` (extends `Settings`, returned to clients), `POSTProviderModel`, `CustomSecretWithoutValueModel`, and `SandboxGroupingStrategy` enum.
- `settings_store.py` + `file_settings_store.py` — abstract store + file backend.
- `settings_router.py` — settings endpoints.
- `llm_profiles.py` — named LLM configuration profiles.

## secrets/
- `secrets_models.py` — secret models.
- `secrets_store.py` + `file_secrets_store.py` — abstract + file backend.
- `secrets_router.py` — endpoints. Git provider tokens go through V1 secrets endpoints: `POST` / `DELETE /api/v1/secrets/git-providers`.

## Adding a new user setting (full checklist in AGENTS.md)
Backend: add to `Settings` model (and `GETSettingsModel`/API shape). Frontend: `Settings` + `ApiSettings` types, `DEFAULT_SETTINGS`, `useSettings`, `useSaveSettings`, UI component, i18n. Both sides must change.

## Frontend save patterns (which UI to mirror)
- **Immediate-save** (secrets, API keys, MCP servers): dedicated mutation hooks, no Save button.
- **Form-based** (LLM/app settings): `isDirty` + `useSaveSettings`.

## Notes / gotchas
- Don't reuse the logout flow to disconnect git tokens — `useLogout` is for real logout (legacy OSS). Use the secrets git-providers endpoints.
- Secrets are masked by default; unmasking requires the dual-auth flow (see [user-and-auth.md](020-user-and-auth.md)).
