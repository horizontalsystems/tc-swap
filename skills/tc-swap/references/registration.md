# Registration

Registration is a one-time step. Skip this if you already have an `agentKey`.

## Register

```
POST https://api.thorchain.org/agent/register
Content-Type: application/json
```

**Body** (all fields optional):

| Field | Type | Description |
|---|---|---|
| name | string | Readable identifier, e.g. `"my-trading-bot"`. Auto-generated if omitted. Lowercase alphanumeric + hyphens, 2–64 chars. |

**Response:**

```json
{
  "agentId": "550e8400-e29b-41d4-a716-446655440000",
  "agentName": "swift-fox-42",
  "agentKey": "a3f9...",
  "message": "Store your agentKey securely — it will not be shown again."
}
```

Store `agentKey` as `TCSWAP_AGENT_KEY`. It is shown only once.

## Check Agent Status

```
GET https://api.thorchain.org/agent/me
X-Agent-Key: $TCSWAP_AGENT_KEY
```

Returns current rate limit, request count, suspension status, and fulfillment stats.
