# amazon-ads MCP Service

HTTP/SSE MCP server for Amazon Advertising API, powered by
[KuudoAI/amazon_ads_mcp](https://github.com/KuudoAI/amazon_ads_mcp)
(`openbridge/amazon-ads-mcp` Docker image).

Part of EPIC-001 — see [docs/roadmap.md](../docs/roadmap.md).

---

## Required environment variables

Set these in `.env` (local) and Railway → Service → Variables (production).

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
| `PORT` | Port to bind (Railway sets this automatically) |

Phase 1 packages: `profiles,sponsored-products,reporting-version-3`

---

## Local development

Run from the **repo root**:

```bash
docker compose -f amazon-ads/docker-compose.yml up
```

The SSE endpoint will be available at `http://localhost:9080/sse`.

---

## Railway deployment

1. In the Railway dashboard, create a new service in the `ncl-ads-mcp` project.
2. Connect the GitHub repo and set the **Root Directory** to `amazon-ads/`.
3. Railway will detect the `Dockerfile` and build automatically.
4. Add all env vars from the table above under Service → Variables.
5. The public HTTPS URL (e.g. `https://amazon-ads-mcp.up.railway.app`) is your MCP endpoint.
6. Register that URL in the n8n Cloud MCP Client tool node (SSE transport).

---

## ADR

Server choice is documented in
[docs/architecture/001-ADR-amazon-ads-mcp-server-choice.md](../docs/architecture/001-ADR-amazon-ads-mcp-server-choice.md).
