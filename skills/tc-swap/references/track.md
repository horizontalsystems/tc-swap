# Tracking Swaps

```
POST $TCSWAP_BASE_URL/v1/track
Content-Type: application/json
X-Agent-Key: $TCSWAP_AGENT_KEY
```

## Request

```json
{
  "provider": "THORCHAIN",
  "hash": "<inbound tx hash>",
  "fromAsset": "BTC.BTC",
  "fromAmount": "0.1",
  "toAsset": "ETH.ETH",
  "toAddress": "0x...",
  "depositAddress": "<inboundAddress from the dry:false quote>",
  "chainId": "bitcoin"
}
```

`provider` is `"THORCHAIN"` or `"MAYACHAIN"`.

| Field | Required | Notes |
|---|---|---|
| `provider` | yes | `"THORCHAIN"` or `"MAYACHAIN"` |
| `fromAsset` | yes | Same value you used in the quote |
| `toAsset` | yes | Same value you used in the quote |
| `toAddress` | yes | Recipient (your `destinationAddress` from the quote) |
| `hash` | strongly recommended | Inbound transaction hash. Required to read on-chain status reliably; without it, the response will be `not_started` |
| `depositAddress` | strongly recommended | Use the `inboundAddress` returned by the `dry: false` quote |
| `chainId` | strongly recommended | Source-chain identifier (e.g. `"bitcoin"`, `"1"` for Ethereum). See [providers.md](providers.md) |
| `fromAmount` | strongly recommended | The exact `sellAmount` from the quote |

**Why the "strongly recommended" fields matter.** When `depositAddress`, `chainId`, and `fromAmount` are all present, the server immediately reconciles the swap with the record it created at quote time, which keeps your fulfillment ratio accurate. If you omit them, the server still returns the correct on-chain status to you, but the internal record is reconciled by a slower background job — a few minutes of delay before your fulfillment count moves.

## Response

```json
{
  "chainId": "1",
  "hash": "FBB00841...",
  "type": "swap",
  "status": "completed",
  "fromAsset": "ETH.ETH",
  "fromAmount": "0.0005",
  "fromAddress": "0x47c382...",
  "toAsset": "BSC.BNB",
  "toAmount": "0.00143868",
  "toAddress": "0x47c382...",
  "meta": { "provider": "THORCHAIN" },
  "legs": [
    {
      "chainId": "1",
      "hash": "FBB00841...",
      "type": "native_send",
      "status": "completed",
      "fromAsset": "ETH.ETH",
      "fromAmount": "0.0005",
      "fromAddress": "0x47c382...",
      "toAsset": "ETH.ETH",
      "toAmount": "0.0005",
      "toAddress": "0x10d15f..."
    },
    {
      "chainId": "thorchain-1",
      "type": "swap",
      "status": "completed",
      "fromAsset": "ETH.ETH",
      "toAsset": "BSC.BNB",
      "...": "..."
    },
    {
      "chainId": "56",
      "hash": "761c763f...",
      "type": "native_send",
      "status": "completed",
      "fromAsset": "BSC.BNB",
      "toAmount": "0.00143868",
      "toAddress": "0x47c382..."
    }
  ]
}
```

Top-level `status` reflects the overall swap. `legs` is the per-chain breakdown — leg `type` is `native_send` (deposit / payout transfer) or `swap` (the protocol swap on THORChain/Mayachain).

A 404 with `{"message": "Swap not found"}` means the hash isn't visible yet — keep polling.

## Statuses

| Status | Meaning |
|---|---|
| `not_started` | Funds not yet received |
| `pending` | Funds received, waiting to process |
| `swapping` | Swap in progress |
| `completed` | Swap finished, funds delivered |
| `refunded` | Swap failed, funds returned |
| `failed` | Swap failed, no refund |
| `unknown` | Status could not be determined |

**Terminal statuses** (stop polling): `completed`, `refunded`, `failed`.

Poll every 1–2 minutes. THORChain and Mayachain swaps typically complete in 5–20 minutes.
