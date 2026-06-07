# SSRF Bypass for VPN/Proxy Environments

## Problem

When running OpenClaw behind a VPN or behind a corporate proxy that performs DNS hijacking, external API endpoints (like `generativelanguage.googleapis.com` for Gemini) may resolve to private/internal IP addresses. This triggers OpenClaw's SSRF (Server-Side Request Forgery) protection, blocking the request:

```
security: blocked URL fetch (url-fetch) targetOrigin=https://generativelanguage.googleapis.com reason=Blocked: resolves to private/internal/special-use IP address
```

## Solution

Set the `OPENCLAW_BYPASS_SSRF` environment variable to bypass SSRF checks for private network addresses.

### Usage

**Development mode:**

```bash
OPENCLAW_BYPASS_SSRF=1 pnpm gateway:watch
```

**Production:**

```bash
OPENCLAW_BYPASS_SSRF=1 pnpm openclaw ...
```

**Using export:**

```bash
export OPENCLAW_BYPASS_SSRF=1
pnpm gateway:watch
```

### Implementation

The bypass is implemented in `src/infra/net/ssrf.ts` in the `shouldSkipPrivateNetworkChecks` function. When `OPENCLAW_BYPASS_SSRF=1`, the function returns `true` early, skipping all SSRF validation checks.

## Files Modified

| File                    | Change                                                                 |
| ----------------------- | ---------------------------------------------------------------------- |
| `src/infra/net/ssrf.ts` | Added environment variable check in `shouldSkipPrivateNetworkChecks()` |
| `src/infra/dotenv.ts`   | Declared `OPENCLAW_BYPASS_SSRF` environment variable                   |

## Security Note

> **Warning:** Only use this bypass in trusted environments (local development, VPN, corporate networks). Bypassing SSRF checks exposes the application to potential Server-Side Request Forgery attacks.

## Related

- SSRF protection module: `src/infra/net/ssrf.ts`
- Fetch guard: `src/infra/net/fetch-guard.ts`
- Provider config: `models.providers.*.request.allowPrivateNetwork` (config-based alternative)

## Date

2026-04-28
