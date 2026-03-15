# CLAUDE.md — Agent Instructions for ads-mcp-servers

## What this project is

A monorepo of HTTP/SSE MCP servers for ad platform integrations, deployed on Railway.
Each subdirectory is an independent service that exposes live ad platform data as MCP tools.
The primary consumer is an **n8n Cloud** agent that uses these as live-data tools alongside a Pinecone historical index.

Railway project name: **ncl-ads-mcp**

---

## Tech stack

| Concern | Choice |
| --- | --- |
| Service containers | Docker (pre-built upstream images where possible) |
| Local scripting / tooling | `uv` (Python) |
| Hosting | Railway (one service per subdirectory) |
| MCP transport | HTTP/SSE — **stdio is not supported by n8n Cloud** |

---

## Repo structure

```text
ads-mcp-servers/
├── CLAUDE.md                        ← this file (tracked in git)
├── .env                             ← secrets (gitignored — never commit)
├── .env.example                     ← placeholder keys (tracked)
├── .gitignore
├── .ai/
│   └── current_plan.md              ← session handoff; update at end of every session
├── docs/
│   ├── roadmap.md                   ← epic ledger; update when status changes
│   ├── PRD.md                       ← product requirements
│   ├── plans/                       ← implementation plans (one file per epic/task)
│   ├── architecture/                ← ADRs: NNN-ADR-{brief-description}.md
│   │   └── rules/                   ← extracted agent rules from ADRs
│   └── rules/                       ← project-wide coding rules
├── amazon-ads/                      ← EPIC-001 (Active)
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── railway.toml
│   └── README.md
├── google-ads/                      ← EPIC-002 (Pending)
│   └── README.md
├── google-analytics/                ← EPIC-003 (Pending)
│   └── README.md
├── meta-ads/                        ← EPIC-004 (Pending)
│   └── README.md
├── tiktok-ads/                      ← EPIC-005 (Pending)
│   └── README.md
└── shared/                          ← shared utilities (Python helpers, etc.)
```

---

## Deployment model

- Each subdirectory (`amazon-ads/`, `google-ads/`, etc.) maps to **one Railway service**.
- Railway is configured to point each service at its subdirectory root.
- Every service gets its own public HTTPS URL from Railway (e.g. `https://amazon-ads-mcp.up.railway.app`).
- That public URL is registered in the n8n Cloud MCP Client tool node.

### n8n Cloud constraint (hard requirement)

n8n Cloud cannot reach `stdio` MCP servers. Every service **must** use HTTP/SSE transport and expose a **public HTTPS URL**. Never configure `TRANSPORT=stdio`.

---

## How to add a new platform

1. Create a new subdirectory following the `amazon-ads/` pattern:

   - `Dockerfile` — pulls the upstream image
   - `docker-compose.yml` — for local testing
   - `railway.toml` — Railway deploy config
   - `README.md` — setup notes and env var list

2. Add all required env var names (empty values) to `.env.example`.
3. Add actual secrets to `.env` locally and to the Railway service's environment variables in the Railway dashboard.
4. Create a new epic in `docs/roadmap.md`.
5. Write an ADR in `docs/architecture/` explaining the server choice.
6. Update `CLAUDE.md` repo structure tree.

---

## Env var conventions

| Context | Where to set |
| --- | --- |
| Local development | `.env` at repo root (gitignored) |
| Railway production | Railway dashboard → Service → Variables |
| Template / docs | `.env.example` at repo root (tracked, no real values) |

Rules:

- Never hardcode secrets in any tracked file.
- `.env` is gitignored — confirm before every commit with `git status`.
- Each service reads env vars injected at container start (via `env_file` in docker-compose or Railway Variables).

---

## Docs conventions

### `docs/plans/`

One markdown file per epic or major task. Name: `EPIC-NNN-{slug}.md`. Describes implementation steps, decisions made, blockers.

### `docs/architecture/` — ADRs

File: `NNN-ADR-{brief-description}.md`. Use when choosing a library, infrastructure pattern, or making a non-obvious technical decision. Follow the ADR template in the project prompt.

### `docs/architecture/rules/`

One rule file per ADR. Contains the single injected constraint an agent must follow. Agents reading this codebase should load all files in this directory.

### `docs/rules/`

Project-wide rules not tied to a specific ADR (e.g. "all services must expose /sse on the default port").

---

## End-of-session checklist

Before ending any working session, update `.ai/current_plan.md` with:

1. What was completed this session
2. Current state of any in-progress work
3. Exact next step the next session should start with
4. Any blockers or open questions

Keep it under 300 words. This file is gitignored — it is local-only context.
