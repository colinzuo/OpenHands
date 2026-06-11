# integrations / git

`openhands/app_server/integrations/`, `git/`

## What it does
Talks to external git/code hosts: **GitHub, GitLab, Bitbucket, Bitbucket Data Center, Azure DevOps, Forgejo, Jira DC**. Repos, branches, suggested tasks, comments, installations, auth.

## Shape
- `service_types.py` — the shared vocabulary:
  - `ProviderType` enum (GITHUB, GITLAB, BITBUCKET, BITBUCKET_DATA_CENTER, ...).
  - `BaseGitService` (ABC) + `GitService` / `InstallationsService` (Protocols) — the contract each provider implements.
  - Domain models: `Repository`, `Branch`, `PaginatedBranchesResponse`, `SuggestedTask`, `Comment`, `UserGitInfo`.
  - Typed errors: `AuthenticationError`, `RateLimitError`, `ProviderTimeoutError`, `ResourceNotFoundError`, `UnknownException`.
- One subdir per provider (`github/`, `gitlab/`, ...), each with a `*ServiceImpl`.
- `provider.py` — dispatches to the right `*ServiceImpl` based on `ProviderType`; reads `AppMode`.
- `protocols/`, `templates/`, `utils.py` — shared helpers.

## git/
- `git_models.py` + `git_router.py` — the git endpoints exposed to clients (thin layer over the integrations services).

## Notes / gotchas
- To add a provider: implement `BaseGitService`/`GitService`, add a `ProviderType`, register in `provider.py`.
- Errors are normalized to the typed exceptions above — handle those, not raw HTTP errors from each host.
- The **enterprise** layer has its own richer integrations (incl. Jira, Linear, Slack, Stripe) under `enterprise/integrations/` — see [enterprise.md](100-enterprise.md). OSS here is git-host focused.
