# Rule: Amazon Ads MCP Server Choice (ADR-001)

When working in `amazon-ads/`, always use the `openbridge/amazon-ads-mcp` Docker image with
`AUTH_METHOD=direct` and `TRANSPORT=http`. Never switch to an alternative server or stdio transport
without superseding this decision with a new ADR.

## Details

- **Docker image:** `openbridge/amazon-ads-mcp:latest`
- **Transport:** `http` (SSE endpoint at `/sse`)
- **Auth method:** `direct` — requires `AMAZON_AD_API_CLIENT_ID`, `AMAZON_AD_API_CLIENT_SECRET`, `AMAZON_AD_API_REFRESH_TOKEN`
- **Phase 1 packages:** `profiles,sponsored-products,reporting-version-3`
- **Source ADR:** `docs/architecture/001-ADR-amazon-ads-mcp-server-choice.md`
