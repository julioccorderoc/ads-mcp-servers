# ADR-001: Amazon Ads MCP Server Choice

- **Date:** 2026-03-14
- **Status:** Accepted

## 1. Context

We need an MCP server that exposes Amazon Advertising API data as tools consumable by an n8n Cloud
agent. The server must support HTTP/SSE transport (n8n Cloud cannot use stdio), be deployable on
Railway via Docker, and be maintainable without deep Amazon Ads API expertise.

Four options were evaluated:

| Option | Description |
| --- | --- |
| KuudoAI/amazon_ads_mcp | Open-source FastMCP-based server, Docker-native, modular packages, active maintenance |
| ppcprophet | SaaS product wrapping Amazon Ads API |
| amekala/Ads-manager-mcp-server | Node.js server, designed for Replit hosting |
| Build from scratch | Custom Python MCP server using `amazon-ads` SDK |

## 2. Decision

Use **KuudoAI/amazon_ads_mcp** (Docker image: `openbridge/amazon-ads-mcp`) deployed on Railway.

Enable these tool packages for Phase 1: `profiles,sponsored-products,reporting-version-3`.

Use `AUTH_METHOD=direct` with operator-supplied OAuth credentials (client ID, client secret,
refresh token). No Openbridge account required.

## 3. Consequences (The "Why")

### Positive

- Maintained open-source project with clear Docker image (`openbridge/amazon-ads-mcp`)
- HTTP/SSE transport supported out of the box (`TRANSPORT=http`) — satisfies the n8n Cloud requirement
- Modular package system (`AMAZON_AD_API_PACKAGES`) lets us start narrow and expand without server changes
- `AUTH_METHOD=direct` uses operator's own Amazon Ads application — no vendor lock-in on auth
- Railway deployment is straightforward: one Dockerfile, one `railway.toml`, env vars via Railway dashboard
- Active CHANGELOG and GitHub issues indicate ongoing maintenance

### Negative

- Dependency on upstream image (`openbridge/amazon-ads-mcp`) — breaking changes require a version pin
- No dedicated `/health` endpoint; Railway health checks must target `/sse` or be skipped
- `reporting-version-3` package scope is broad; tool list may exceed n8n's MCP tool limit if many packages are enabled simultaneously
- **`Amazon-Advertising-API-ClientId` is not auto-injected.** The server has `AMAZON_AD_API_CLIENT_ID` in env but does not inject it into tool calls. Every `rp_*` and `ac_*` call must include it explicitly. This is a FastMCP schema-generation issue: it mirrors the upstream OpenAPI header parameter as a required tool argument instead of filling it from env. Mitigation: hardcode the value in the n8n system prompt (client ID is a public OAuth app identifier, not a secret, but it still appears in prompt logs). Proper fix: monkey-patch the generated tool schemas at server startup to remove `Amazon-Advertising-API-ClientId` from the `required` array and inject it server-side — tracked in EPIC-001 Known Limitations.
- **`download_export` is unusable for HTTP/SSE deployments.** The tool downloads the S3 report file to the Railway container filesystem (`/app/data/reports/s3-reports/`). There is no MCP tool or resource that reads file contents back to the client. In a local stdio deployment the Python process can read local files directly; in our HTTP/SSE Railway deployment the file is stranded. Workaround: the n8n workflow must fetch the S3 pre-signed URL directly via an HTTP Request node and pass the decompressed JSON back to the AI Agent — the agent should never call `download_export`. Tracked in EPIC-001 Known Limitations.

## 4. Rule Extraction (The "How" for Agents)

- **Target File:** `docs/architecture/rules/001-ADR-amazon-ads-mcp-server-choice.md`
- **Injected Constraint:** When working in `amazon-ads/`, always use the `openbridge/amazon-ads-mcp`
  Docker image with `AUTH_METHOD=direct` and `TRANSPORT=http`; never switch to an alternative server
  or stdio transport without a new ADR.
