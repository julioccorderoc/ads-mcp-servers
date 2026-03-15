# amazon-ads MCP Service

MCP server for Amazon Advertising API, powered by
[KuudoAI/amazon_ads_mcp](https://github.com/KuudoAI/amazon_ads_mcp)
(`openbridge/amazon-ads-mcp` Docker image).

- **Transport:** Streamable HTTP (MCP spec 2025-03-26)
- **Endpoint:** `/mcp`
- **Part of:** EPIC-001 — see [docs/roadmap.md](../docs/roadmap.md)
- **Security audit:** [docs/security-review-amazon-ads-mcp.md](../docs/security-review-amazon-ads-mcp.md)

---

## Required environment variables

Set these in `.env` (local) and Railway → Service → Variables (production).
See [.env.example](../.env.example) for the full template.

| Variable | Description |
| --- | --- |
| `AMAZON_AD_API_CLIENT_ID` | Amazon Ads application client ID |
| `AMAZON_AD_API_CLIENT_SECRET` | Amazon Ads application client secret |
| `AMAZON_AD_API_REFRESH_TOKEN` | OAuth refresh token |
| `AMAZON_ADS_PROFILE_ID` | Default profile/scope ID |
| `AMAZON_ADS_REGION` | Region: `na`, `eu`, or `fe` |
| `AMAZON_AD_API_PACKAGES` | Comma-separated tool packages to load |
| `AUTH_METHOD` | Set to `direct` |
| `TRANSPORT` | Set to `http` |
| `PORT` | Port to bind (Railway injects this automatically) |

Phase 1 packages: `profiles,sponsored-products,reporting-version-3`

---

## Local development

Run from the **repo root**:

```bash
docker compose -f amazon-ads/docker-compose.yml up
```

The MCP endpoint is at `http://localhost:9080/mcp`.

### Smoke test (3-step MCP handshake)

```bash
# 1. Initialize — captures mcp-session-id from response headers
curl -v http://localhost:9080/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"0.1"}}}' \
  2>&1 | grep -E "mcp-session-id|jsonrpc"

# 2. Send initialized notification (no response body expected)
curl -s http://localhost:9080/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "mcp-session-id: PASTE_SESSION_ID" \
  -d '{"jsonrpc":"2.0","method":"notifications/initialized"}'

# 3. List tools
curl -s http://localhost:9080/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "mcp-session-id: PASTE_SESSION_ID" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'
```

---

## Railway deployment

1. In the Railway dashboard, create a new service in the `ncl-ads-mcp` project.
2. Connect the GitHub repo and set the **Root Directory** to `amazon-ads/`.
3. Railway detects the `Dockerfile` and builds automatically.
4. Add all env vars from the table above under Service → Variables.
   Railway injects `PORT` automatically — do not set it manually.
5. Deploy and confirm the build succeeds.
6. The public HTTPS URL (e.g. `https://amazon-ads-mcp-production.up.railway.app/mcp`) is your MCP endpoint.
7. In n8n Cloud, add an MCP Client tool node with **Streamable HTTP** transport pointing to that URL.

---

## ADR

Server choice documented in
[docs/architecture/001-ADR-amazon-ads-mcp-server-choice.md](../docs/architecture/001-ADR-amazon-ads-mcp-server-choice.md).
