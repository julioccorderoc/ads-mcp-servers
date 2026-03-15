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

## 4. Rule Extraction (The "How" for Agents)

- **Target File:** `docs/architecture/rules/001-ADR-amazon-ads-mcp-server-choice.md`
- **Injected Constraint:** When working in `amazon-ads/`, always use the `openbridge/amazon-ads-mcp`
  Docker image with `AUTH_METHOD=direct` and `TRANSPORT=http`; never switch to an alternative server
  or stdio transport without a new ADR.
