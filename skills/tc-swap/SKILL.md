---
name: tc-swap
description: Use this skill to get cryptocurrency swap quotes, execute cross-chain swaps, and track swap status via THORChain and Mayachain.
version: "1.0.0"
metadata:
  author: Horizontal Systems
  homepage: https://swap.thorchain.org
  openclaw:
    emoji: ⚡
    primaryEnv: TCSWAP_AGENT_KEY
    requires:
      env:
        - TCSWAP_AGENT_KEY
        - TCSWAP_BASE_URL
---

# TC Swap — THORChain & Mayachain

All requests require `X-Agent-Key: $TCSWAP_AGENT_KEY` header.  
Base URL: `$TCSWAP_BASE_URL` (e.g. `https://api.thorchain.org/agent`)

## Task Routing

| Task | Reference |
|---|---|
| First-time setup, get an agent key | [registration.md](references/registration.md) |
| Get quotes, pick a route, execute a swap | [quote.md](references/quote.md) |
| Track swap status after funds are sent | [track.md](references/track.md) |
| Providers, supported chains, asset ID format | [providers.md](references/providers.md) |

## Swap Flow (Summary)

1. **Quote (dry)** — `POST /v1/quote` with `dry: true` to compare routes
2. **Quote (real)** — `POST /v1/quote` with `dry: false` + one provider to create an order
3. **Send funds** — send `sellAmount` of `sellAsset` to `inboundAddress` with `memo` included
4. **Track** — `POST /v1/track` until status is `completed`, `refunded`, or `failed`

## Error Codes

| Code | Meaning |
|---|---|
| 401 | Missing or invalid `X-Agent-Key` |
| 403 | Agent suspended |
| 409 | Agent name already taken |
| 429 | Rate limit exceeded |
| 400 | Validation error — check response body |
| 404 | No routes found for the requested swap |
