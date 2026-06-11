# frontend

`frontend/` — React SPA (Vite, React Router, TanStack Query, Tailwind, vitest).

## Strict layering (do not violate)
```
components / routes  →  hooks/{query,mutation}  →  api  →  backend REST
```
- `src/api` — Data Access Layer. **Never** called directly from components.
- `src/hooks/query/` — `use[Resource]` read hooks (e.g. `useConversationSkills`).
- `src/hooks/mutation/` — `use[Action]` write hooks (e.g. `useDeleteConversation`).

## Layout (`frontend/src/`)
- `routes/` + `routes.ts` — pages (React Router); `root.tsx`, `entry.client.tsx` — app shell.
- `components/`, `ui/` — presentational + design-system components.
- `stores/`, `context/`, `contexts/` — client state (note: both `context` and `contexts` exist).
- `services/` — non-API client logic (e.g. `services/settings.ts` holds `DEFAULT_SETTINGS`).
- `types/` — TS types incl. `settings.ts`, `action-type.ts`.
- `i18n/` — `translation.json` + `declaration.ts` (regen with `npm run make-i18n`).
- `mocks/`, `__mocks__/`, `__tests__/`, `tests/` — vitest + mock data.
- `query-client-config.ts` — TanStack Query client setup.

## Two settings-save patterns
- **Immediate-save**: entity resources (API keys, secrets, MCP servers) — dedicated mutation hooks, no Save button.
- **Form-based**: LLM/app settings — local `isDirty` + `useSaveSettings`.

## Action display
- `HANDLED_ACTIONS` in `state/chat-slice.ts` decides which action types render as collapsible UI; handler in `addAssistantAction`. New action type ⇒ add to array + handler + `ACTION_MESSAGE$NAME` i18n key.

## Commands (run in `frontend/`)
`npm run test` · `npm run test -- -t "Name"` · `npm run build` · `npm run lint:fix` · `npm run make-i18n`

## Notes / gotchas
- When mocking `useAgentState` in tests, always include `isArchived` (else archived-conversation UI breaks). See AGENTS.md.
- For SaaS org screens, derive the selected org from `useOrganizations()` + selected-org-id store rather than a dedicated single-org fetch.
