# ads-mcp-servers

Monorepo of HTTP MCP servers for ad platform integrations, deployed on Railway.
Each subdirectory is an independent service consumed by an **n8n Cloud** agent as a live-data tool.

Railway project: **ncl-ads-mcp**

---

## Services

| Service | Status | Transport | Docs |
| --- | --- | --- | --- |
| [amazon-ads/](amazon-ads/) | Active (EPIC-001) | Streamable HTTP `/mcp` | [README](amazon-ads/README.md) |
| [google-ads/](google-ads/) | Pending (EPIC-002) | — | [README](google-ads/README.md) |
| [google-analytics/](google-analytics/) | Pending (EPIC-003) | — | [README](google-analytics/README.md) |
| [meta-ads/](meta-ads/) | Pending (EPIC-004) | — | [README](meta-ads/README.md) |
| [tiktok-ads/](tiktok-ads/) | Pending (EPIC-005) | — | [README](tiktok-ads/README.md) |

---

## Architecture

- One Railway service per subdirectory, each with its own public HTTPS URL
- All services use Streamable HTTP MCP transport (`/mcp` endpoint, MCP spec 2025-03-26)
- n8n Cloud connects to each service via its Railway URL
- Secrets live in `.env` locally and Railway → Service → Variables in production — never committed

See [CLAUDE.md](CLAUDE.md) for full agent instructions and [docs/roadmap.md](docs/roadmap.md) for the epic ledger.

---

## Quick start (local)

Prerequisites: Docker Desktop running.

```bash
# Amazon Ads
docker compose -f amazon-ads/docker-compose.yml up
```

MCP endpoint: `http://localhost:9080/mcp`

---

## Docs

| File | Purpose |
| --- | --- |
| [CLAUDE.md](CLAUDE.md) | Agent instructions, conventions, env var rules |
| [AGENT.md](AGENT.md) | n8n agent system prompt (Paid Ads Associate) |
| [docs/roadmap.md](docs/roadmap.md) | Epic ledger |
| [docs/architecture/](docs/architecture/) | ADRs |
| [docs/architecture/rules/](docs/architecture/rules/) | Injected agent constraints |
| [docs/security-review-amazon-ads-mcp.md](docs/security-review-amazon-ads-mcp.md) | Security audit of amazon-ads image |
| [.env.example](.env.example) | Env var template (no real values) |
