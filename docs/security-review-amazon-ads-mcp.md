# Security Review: openbridge/amazon-ads-mcp

- **Date:** 2026-03-14
- **Reviewed by:** Claude (automated source + image audit)
- **Verdict:** YELLOW-GREEN — safe to deploy with current configuration
- **Action required:** None blocking. See recommendations for hardening.

---

## What was reviewed

| Artifact | Path |
| --- | --- |
| GitHub source | `github.com/KuudoAI/amazon_ads_mcp` |
| Docker image | `openbridge/amazon-ads-mcp:latest` |
| Auth flow | `src/amazon_ads_mcp/auth/providers/direct.py` |
| Token storage | `src/amazon_ads_mcp/auth/token_store.py` |
| Auth manager | `src/amazon_ads_mcp/auth/manager.py` |
| HTTP endpoints | `src/amazon_ads_mcp/utils/region_config.py` |
| HTTP client | `src/amazon_ads_mcp/utils/http_client.py` |

---

## Key findings

### Credential handling — CLEAN

With `AUTH_METHOD=direct`, your credentials only ever leave the container to reach Amazon's own servers:

| Call | Destination |
| --- | --- |
| OAuth token refresh | `https://api.amazon.com/auth/o2/token` (NA) |
| API requests | `https://advertising-api.amazon.com` (NA) |

No third-party server (including Openbridge) ever receives your `client_id`, `client_secret`, or `refresh_token` in the `direct` auth path.

The `openbridge.*` code paths (`OPENBRIDGE_AUTH_BASE_URL`, etc.) are only exercised when `AUTH_METHOD=openbridge` — which we are **not** using.

### Token storage — CLEAN

- Access tokens: in-memory only (ephemeral)
- Refresh tokens: encrypted on disk with Fernet (AES-128) when `AMAZON_ADS_TOKEN_PERSIST=true`
- Encryption key is generated locally and stored at `/app/.cache/amazon-ads-mcp/.encryption.key`
- No remote token storage

### KuudoAI / Openbridge identity — EXPLAINED

KuudoAI is a GitHub Organization associated with **Openbridge**, a legitimate AWS Partner company that provides Amazon Advertising data pipelines. They have been operating for years with millions of Docker Hub pulls on other images (`ob_bulkstash`: 1.5M+ pulls). The "ad SaaS" connection is Openbridge itself — not a red flag.

---

## Residual concerns

| Concern | Risk level | Notes |
| --- | --- | --- |
| Pre-built Docker Hub image | Low | Can't verify image ≡ source; Dockerfile is simple and transparent |
| `openai>=1.109.1` dependency | Low | Used only by the optional `CODE_MODE` feature, not credential handling |
| No third-party security audit | Low | Code is readable and auditable but not externally certified |
| Auto-generated encryption key stored alongside tokens | Low | Key lives in the same volume as the tokens it encrypts |
| Repo created September 2025 | Low | New, but consistent with MCP emerging in 2025; Openbridge is established |

---

## Recommendations for future sessions

These are non-blocking improvements, ordered by impact:

### 1. Build the image from source (eliminates Docker Hub trust concern)

Replace the single `FROM` line in `amazon-ads/Dockerfile`:

```dockerfile
# Option A: Build directly from source
# Clone https://github.com/KuudoAI/amazon_ads_mcp, copy into amazon-ads/src/,
# and use their Dockerfile as-is.

# Option B: Pin to a specific digest instead of :latest
FROM openbridge/amazon-ads-mcp@sha256:<specific-digest>
```

Pinning to a digest prevents silent image replacement.

### 2. Set an explicit encryption key for token persistence

Add to Railway Variables (never commit):

```bash
# Generate with:
# python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
AMAZON_ADS_ENCRYPTION_KEY=<generated-key>
```

This separates the encryption key from the encrypted data. Without it, the auto-generated key is stored alongside the tokens it protects.

### 3. Run the container read-only with explicit volume mounts

```yaml
# In docker-compose.yml
security_opt:
  - no-new-privileges:true
read_only: true
tmpfs:
  - /tmp
```

The volumes already isolate data; adding read-only prevents any writes to the container filesystem.

### 4. Pin the image digest in CI

Once a known-good version is verified, pin it in `Dockerfile` and `docker-compose.yml`:

```bash
docker inspect openbridge/amazon-ads-mcp:latest --format '{{index .RepoDigests 0}}'
```

---

## Not investigated

- The `openbridge.py` provider (only relevant if `AUTH_METHOD=openbridge`)
- `CODE_MODE` execution sandbox (not enabled in our config)
- Full `http_client.py` (>35KB; reviewed structure, not every method)
- `server/mcp_server.py` and `server/server_builder.py` (tool registration logic)

If `AUTH_METHOD` or `CODE_MODE` are ever changed, re-audit the relevant provider.
