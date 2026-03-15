# ROADMAP

- **Version:** 0.1
- **Last Updated:** 2026-03-14
- **Primary Human Owner:** Julio

## Operating Rules for the Planner Agent

1. You may only move one Epic to `Active` at a time.
2. Before marking an Epic `Complete`, you must verify all its Success Criteria are met in the main branch.
3. Do not parse or extract Epics that depend on incomplete prerequisites.

## Epic Ledger

### EPIC-001

- **Status:** Active
- **Dependencies:** None
- **Business Objective:** Enable the n8n agent to query live Amazon Ads data
- **Technical Boundary:** `amazon-ads/` service deployed on Railway, reachable via HTTPS/SSE from n8n Cloud
- **Verification Criteria (Definition of Done):**
  - Railway service passes health check
  - n8n Cloud MCP Client tool node can call at least one tool successfully
  - No credentials committed to git

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
