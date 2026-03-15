# ROADMAP

- **Version:** 0.2
- **Last Updated:** 2026-03-15
- **Primary Human Owner:** Julio

## Operating Rules for the Planner Agent

1. You may only move one Epic to `Active` at a time.
2. Before marking an Epic `Complete`, you must verify all its Success Criteria are met in the main branch.
3. Do not parse or extract Epics that depend on incomplete prerequisites.

## Epic Ledger

### EPIC-001

- **Status:** Complete (deployed 2026-03-15)
- **Dependencies:** None
- **Business Objective:** Enable the n8n agent to query live Amazon Ads data
- **Technical Boundary:** `amazon-ads/` service deployed on Railway, reachable via HTTPS from n8n Cloud
- **Production URL:** `https://amazon-ads-production.up.railway.app/mcp`
- **Active packages:** `profiles,reporting-version-3`
- **Verification Criteria (Definition of Done):**
  - [x] Railway service passes health check
  - [ ] n8n Cloud MCP Client tool node can call at least one tool successfully
  - [x] No credentials committed to git
- **Known limitations:**
  - `sponsored-products` package excluded due to n8n MCP client bug with nested JSON Schema
    `$defs` (see [amazon-ads/README.md](../amazon-ads/README.md))
  - `Amazon-Advertising-API-ClientId` not auto-injected by server — must be passed explicitly in every `rp_*` / `ac_*` tool call; workaround is hardcoding in system prompt; proper fix requires patching the upstream image schema at startup
  - `download_export` unusable for HTTP/SSE: file stored on Railway container with no read-back mechanism; n8n workflow must fetch the S3 URL directly via HTTP Request node instead

### EPIC-002

- **Status:** Pending
- **Dependencies:** EPIC-001
- **Business Objective:** Enable the n8n agent to query live Google Ads data
- **Technical Boundary:** `google-ads/` service deployed on Railway, reachable via HTTPS/SSE
- **Verification Criteria (Definition of Done):**
  - Railway service passes health check
  - n8n Cloud MCP Client tool node can call at least one tool successfully
  - No credentials committed to git

### EPIC-003

- **Status:** Pending
- **Dependencies:** EPIC-001
- **Business Objective:** Enable the n8n agent to query Google Analytics (GA4) data
- **Technical Boundary:** `google-analytics/` service deployed on Railway, reachable via HTTPS/SSE
- **Verification Criteria (Definition of Done):**
  - Railway service passes health check
  - n8n Cloud MCP Client tool node can call at least one tool successfully
  - No credentials committed to git

### EPIC-004

- **Status:** Pending
- **Dependencies:** EPIC-001
- **Business Objective:** Enable the n8n agent to query live Meta Ads data (Facebook/Instagram)
- **Technical Boundary:** `meta-ads/` service deployed on Railway, reachable via HTTPS/SSE
- **Verification Criteria (Definition of Done):**
  - Railway service passes health check
  - n8n Cloud MCP Client tool node can call at least one tool successfully
  - No credentials committed to git

### EPIC-005

- **Status:** Pending
- **Dependencies:** EPIC-001
- **Business Objective:** Enable the n8n agent to query live TikTok Ads data
- **Technical Boundary:** `tiktok-ads/` service deployed on Railway, reachable via HTTPS/SSE
- **Verification Criteria (Definition of Done):**
  - Railway service passes health check
  - n8n Cloud MCP Client tool node can call at least one tool successfully
  - No credentials committed to git
