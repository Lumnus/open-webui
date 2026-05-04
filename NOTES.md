# NOTES — Lumnus fork of Open WebUI

Scratch + source-walk + decisions for the Lumnus fork. Lives on `lumnus` branch only (not on `main`, which exact-tracks `upstream/main`).

**Fork home:** `Lumnus/open-webui` on GitHub.
**Local clone:** `~/Git/open-webui/`
**Active branch:** `lumnus`
**Run folder:** [`~/.claude/runs/2026-W18/2026-05-04-openwebui-fork-firstwalk/`](~/.claude/runs/2026-W18/2026-05-04-openwebui-fork-firstwalk/)
**Why this exists:** test the *fork-and-learn-through-the-app's-logic* mode of working — clone, run, source-walk, modify, test, enhance — with a real production AI app codebase. Operator framing 2026-05-04: *"It's an interesting thing to do. I wanted to test this way of working for some time."*

---

## Branch shape + sync discipline

```
upstream (open-webui/open-webui)
    │
    ▼ git fetch upstream && git rebase upstream/main main
    │
   main  (exact tracking of upstream/main; never directly modified locally)
    │
    ▼ git checkout lumnus && git rebase main && git push --force-with-lease
    │
   lumnus  (active modification branch; default deploy target)
    │
    ▼ short-lived feature branches
    │
   feature/<name> (squash-merge back to lumnus, delete)
```

Sync cadence: **weekly initially**. Conflicts resolved as they arrive (never accumulated).

---

## Stack quick-map

- **Backend:** Python / FastAPI / hatch build / pyproject.toml. Backend lives in `backend/` (when present after build).
- **Frontend:** SvelteKit (Vite, TypeScript). `src/` is the Svelte app, `package.json` / `vite.config.ts` at root.
- **Container:** Single multi-stage `Dockerfile` builds both into one image.
- **Tests:** Cypress for e2e (`cypress.config.ts`); pytest for backend (look in `backend/tests/`); vitest probably for frontend.
- **i18n:** i18next, `i18next-parser.config.ts` at root.

## What we want to learn (open questions, captured as we walk)

1. **Conversation persistence schema** — DB tables, message format, attachment handling, how chats relate to users / sessions / models.
2. **Auth flow** — built-in user model, OIDC support, where in code the auth boundary sits, what the Access JWT trust shape looks like.
3. **Model integration** — how OW discovers OpenAI-compat endpoints, the `/v1/models` consumption, model selection UI, per-model defaults.
4. **Streaming** — SSE? WebSocket? How does it hold up through cloudflared + nginx-ingress?
5. **Tool / function calling** — does OW expose hooks for MCP tools or substrate retrieval?
6. **RAG / file upload** — vector store choice, embedding model, can we point at substrate-pg?
7. **Multi-user** — workspace / chat-share semantics; is it real-time or eventual?
8. **i18n** — easy place to land the first cosmetic patch (validate the deploy loop works).
9. **Settings / admin UI** — what's exposed to users vs admins; how is the admin user determined?
10. **Where does `WEBUI_NAME` / theming / branding live** — for Lumnus skin first walk.

## Lumnus patches — registry

Empty for now; populate as patches land on `lumnus`. Each entry:
- **Patch name** — short slug
- **Why** — what problem it solves for our use case
- **Files touched** — concrete paths
- **Upstream-PR-able?** — yes (generic improvement) / no (Lumnus-specific)

## Gaps identified — running list

1. **Conversation history is OW-DB-side, not JSONL-attached** — known coming in (Phase A2 of [`PENDING_OPEN_WEBUI_SESSION_ATTACH`](~/.claude/PENDING_OPEN_WEBUI_SESSION_ATTACH.md)). Bridging is the substantive fork-level patch.
2. *(more accrete here as we walk)*

## Source-walk log

Each source-walk session appends a dated entry: what files were read, surprising patterns, design choices to note, places to revisit, hypotheses about modification points.

### 2026-05-04 — quick-map landed; first deploy uses upstream image

Just initial orientation. Full source-walk happens after first-deploy gives us a live instance to poke at. Confirmed: SvelteKit frontend, FastAPI backend, single-image Dockerfile, multi-language i18n. Most active branches: `main` (release) + `dev`. Tagged release `0.9.2`.

---

## How to run locally (when needed)

Per upstream README, two paths:
1. `docker compose up` (their reference compose) — simplest.
2. Native dev: backend `cd backend && python -m open_webui.main`, frontend `npm install && npm run dev`. Slower setup, faster feedback loop.

For Lumnus first-walk we run **in-cluster only** (per operator framing *"launch it in the cluster"*). Local dev surfaces if/when source-walk needs faster iteration than rebuild → push → deploy.

## Cross-references

1. [`~/.claude/PENDING_OPEN_WEBUI_SESSION_ATTACH.md`](~/.claude/PENDING_OPEN_WEBUI_SESSION_ATTACH.md) — substantive fork-level patch (deferred until source-walk gives shape).
2. [`~/Git/Hub/infra/k3s-forge/apps/openwebui/`](~/Git/Hub/infra/k3s-forge/apps/openwebui/) — cluster manifests (added in this run).
3. [`~/.claude/runs/2026-W18/2026-05-04-openwebui-fork-firstwalk/`](~/.claude/runs/2026-W18/2026-05-04-openwebui-fork-firstwalk/) — run folder.
