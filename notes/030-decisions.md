# Decisions

Non-trivial choices go here, append-only. Prevents second-guessing, re-litigating old debates, and undoing past work. Newest first.

Format:
```
## YYYY-MM-DD — <decision>
**Context:** why this came up
**Decision:** what we chose
**Rationale:** why
**Alternatives rejected:** what we didn't do, and why
```

---

## 2026-06-11 — Treat this repo as the GUI/Cloud layer, not the agent engine
**Context:** When reading the code, the local `openhands/` tree is much smaller than older `AGENTS.md` text implies; references to `openhands/sdk/...` don't resolve.
**Decision:** Document explicitly (in CLAUDE.md and 020-architecture.md) that agent/LLM/runtime logic lives in the separate `software-agent-sdk` repo, consumed via the `openhands-agent-server` pip dependency.
**Rationale:** Saves future-me from hunting for SDK files that aren't here.
**Alternatives rejected:** Copying SDK internals into notes — they live in another repo and would go stale.

## 2026-06-11 — CLAUDE.md is a quick-start that defers to AGENTS.md
**Context:** A detailed 27 KB `AGENTS.md` already exists.
**Decision:** Keep CLAUDE.md short, point to AGENTS.md for deep dives rather than duplicating.
**Rationale:** Avoid two sources of truth drifting apart.
