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

**Active packages (production):** `profiles,reporting-version-3`

> **Why not `sponsored-products`?** The `sponsored-products` package exposes 80+ tools with
> deeply nested `$defs`/`$ref` schemas. n8n Cloud's MCP client internally converts JSON Schema
> `$defs` to OpenAPI `components/schemas`, and the conversion produces dangling references
> (e.g. `#/components/schemas/BidAnalysesPerPlacement`) that fail schema validation, blocking
> the entire tool list from loading. Until n8n fixes this, keep SP excluded.
> If you need SP write tools, use a direct MCP client that handles JSON Schema `$defs` natively.

---

## Available tools (19 total)

| Prefix | Tool | Description |
| --- | --- | --- |
| `ac_` | `listProfiles` | List all ad profiles on the account |
| `ac_` | `getProfileById` | Fetch a single profile by ID |
| `ac_` | `updateProfiles` | Update profile metadata |
| `rp_` | `createAsyncReport` | Submit an async report request |
| `rp_` | `getAsyncReport` | Poll report status / get download URL |
| `rp_` | `deleteAsyncReport` | Cancel a pending report |
| built-in | `set_active_profile` / `get_active_profile` | Set default profile for subsequent calls |
| built-in | `set_region` / `get_region` | Switch API region at runtime |
| built-in | `download_export` / `list_downloads` | Download completed report files |
| built-in | `start_oauth_flow` / `check_oauth_status` / `refresh_oauth_token` / `clear_oauth_tokens` | OAuth helpers |

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

### Initial setup (one-time)

1. In the Railway dashboard, open the `ncl-ads-mcp` project → `amazon-ads` service.
2. Under **Settings → Source**, set **Root Directory** to `amazon-ads`.
   This is required — without it Railway builds from the repo root, picks up
   the top-level `pyproject.toml`/`uv.lock`, and Railpack ignores the Dockerfile.
3. Add all env vars from the table above under Service → Variables.
   Do **not** set `PORT` or `HOST` — Railway injects `PORT` automatically.

### Re-deploying manually

Railway auto-deploys on every push to `master` once the root directory is set.
For a manual deploy (e.g. from a machine without the repo):

```bash
# Run from the repo root
railway up --service amazon-ads --ci --path-as-root amazon-ads
```

> **Why `--path-as-root`?** `railway up` archives from the git root by default, which
> causes Railpack to detect Python instead of the Dockerfile. This flag scopes the
> archive to the `amazon-ads/` subdirectory only.

### How `$PORT` binding works

The upstream `openbridge/amazon-ads-mcp` image has a hardcoded `CMD` with `--port 9080`.
Railway's `startCommand` (from `railway.toml`) is passed in exec form — no shell — so
`$PORT` would be received as a literal string and the server would crash.

The fix is in [`Dockerfile`](Dockerfile): the CMD is overridden in **shell form**:

```dockerfile
CMD python -m amazon_ads_mcp.server --transport http --host 0.0.0.0 --port ${PORT:-9080}
```

Shell form means Docker wraps the command in `/bin/sh -c`, which expands `$PORT` at
container startup. The `railway.toml` intentionally has **no** `startCommand` so this
shell-form CMD is used instead of a Railway override (which would be exec form).

### n8n Cloud wiring

- Transport: **HTTP Streamable**
- URL: `https://amazon-ads-production.up.railway.app/mcp`
- Authentication: None

---

## ADR

Server choice documented in
[docs/architecture/001-ADR-amazon-ads-mcp-server-choice.md](../docs/architecture/001-ADR-amazon-ads-mcp-server-choice.md).
