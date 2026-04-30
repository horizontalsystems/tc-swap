# Quotes & Swap Execution

## Step 1 — Dry Quote (compare routes)

```
POST $TCSWAP_BASE_URL/v1/quote
Content-Type: application/json
X-Agent-Key: $TCSWAP_AGENT_KEY
```

```json
{
  "sellAsset": "BTC.BTC",
  "buyAsset": "ETH.ETH",
  "sellAmount": "0.1",
  "slippage": 3,
  "destinationAddress": "0x...",
  "refundAddress": "bc1q...",
  "sourceAddress": "bc1q..."
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| sellAsset | string | yes | Asset to sell — see [providers.md](providers.md) for format |
| buyAsset | string | yes | Asset to buy |
| sellAmount | string | yes | Decimal string, e.g. `"0.1"` |
| slippage | number | yes | Tolerance 0–99 (e.g. `3` = 3%) |
| destinationAddress | string | no | Receiving address |
| refundAddress | string | no | Refund address on failure. **If omitted, refunds go to the sender** (the address that funded the inbound tx). |
| sourceAddress | string | no | Sending address. **Required for EVM source chains** to receive a prepared `tx` object in the response — see [Step 3](#step-3--send-funds). |
| providers | string[] | no | Filter to `["THORCHAIN"]` or `["MAYACHAIN"]`. If omitted, both are queried. |
| dry | boolean | no | Default `true`. `false` = create real order. |
| streamingInterval | number | no | THORChain only. Blocks between sub-swaps (1 block ≈ 6 s). See [Streaming Swaps](#streaming-swaps-thorchain) below. |
| streamingQuantity | number | no | THORChain only. Number of sub-swaps. See [Streaming Swaps](#streaming-swaps-thorchain) below. |

**Response:**

```json
{
  "routes": [
    {
      "sellAsset": "BTC.BTC",
      "buyAsset": "ETH.ETH",
      "sellAmount": "0.1",
      "expectedBuyAmount": "1.52",
      "inboundAddress": "bc1q...",
      "destinationAddress": "0x...",
      "fees": [{ "type": "liquidity", "asset": "BTC.BTC", "amount": "0.0003" }],
      "estimatedTime": { "inbound": 600, "swap": 60, "outbound": 600, "total": 1260 },
      "providers": ["THORCHAIN"],
      "memo": "=:ETH.ETH:0x..."
    }
  ],
  "providerErrors": []
}
```

Pick the route with the best `expectedBuyAmount`.

> ⚠️ **Dry quotes are illustrative only.** `inboundAddress`, `memo`, and any `tx` field returned with `dry: true` are placeholders for route comparison — they are **not** valid for sending funds. Sending to a dry-quote `inboundAddress` will result in lost funds with no refund. Always re-issue the quote with `dry: false` and use the addresses/memo from that response.

## Step 2 — Real Quote (create order)

Call the same endpoint with `dry: false` and exactly one provider:

```json
{
  "sellAsset": "BTC.BTC",
  "buyAsset": "ETH.ETH",
  "sellAmount": "0.1",
  "slippage": 3,
  "destinationAddress": "0x...",
  "providers": ["THORCHAIN"],
  "dry": false
}
```

## Step 3 — Send Funds

Send exactly `sellAmount` of `sellAsset` to `inboundAddress` and include the `memo` from the route. Omitting the memo causes a refund — you pay network fees twice.

How the memo travels depends on the source chain:

- **EVM chains (ETH, BSC, AVAX, BASE, ARB)** — the route includes a prepared `tx` object: `{ to, from, value, data, gasPrice, gas? }`. `to` is the THORChain/Mayachain router contract (not `inboundAddress`); the memo is already encoded into `data`. Sign and broadcast `tx` as-is. `inboundAddress` is informational only.
  > **`tx` is only populated when `sourceAddress` is supplied in the quote request.** Without `sourceAddress` the response omits `tx` and the agent has no ready-to-sign transaction. Always pass `sourceAddress` when the source chain is EVM.
- **UTXO chains (BTC, BCH, LTC, DOGE, DASH, ZEC)** — **build the transaction yourself**. Do not use the `tx` field if present; it's not reliable for real wallets (covers only a single address's UTXOs). Construct a transaction with two outputs:
  1. `sellAmount` to `inboundAddress`
  2. an `OP_RETURN` output containing the `memo` string

  Select inputs from all of the sender's addresses as needed, and send change back to the sender.
- **Cosmos / THOR / MAYA / others** — send `sellAmount` to `inboundAddress` with `memo` in the standard memo field.

### Mayachain + ZCash

When selling `ZEC.ZEC` via Mayachain, the route may include a `shielded_memo_address`. Both outputs must be in the **same transaction**: send the funds amount to `inboundAddress` (transparent) and send the memo text to `shielded_memo_address` (shielded). Shielded addresses support longer memos than transparent outputs, which is required for Mayachain's full memo format. Splitting into separate transactions will not work.

## Step 4 — Track

See [track.md](track.md).

## Streaming Swaps (THORChain)

THORChain natively splits large swaps into many sub-swaps over time to reduce slippage. Two parameters control this — both apply only when `providers: ["THORCHAIN"]` (Mayachain ignores them).

### `streamingQuantity` — number of sub-swaps

| Value | Behavior |
|---|---|
| `0` *(recommended default — also the result of omitting it)* | Network auto-calculates the optimal count from the swap size and the per-sub-swap minimum slip-fee floor (5 bps). Best for most agents. |
| `≥ 1` | Explicit count. Capped network-side at `STREAMINGSWAPMAXLENGTH` (~14,400 blocks ≈ 24 h of total streaming). |

### `streamingInterval` — blocks between sub-swaps (1 block ≈ 6 s)

| Value | Mode | Behavior |
|---|---|---|
| `0` | **Rapid** | Multiple sub-swaps within a single block. Fastest total time, but less price improvement — arbitrageurs don't get to rebalance the pool between sub-swaps. |
| `≥ 1` | **Traditional streaming** | One sub-swap every N blocks. Higher N → more time for arbitrage → better effective price, but longer total time. |

### Tradeoff cheat-sheet

| Use case | `streamingQuantity` | `streamingInterval` |
|---|---|---|
| Small swap, want speed | omit (or `0`) | `0` (rapid) |
| Large swap, want best price | `0` (auto) | `1`–`10` |
| Manual control | explicit `N` | explicit `N` |

Approximate price improvement vs a single-shot swap is `~(quantity − 1) / quantity` of the slippage saved — e.g. 10 sub-swaps capture ~90%.

The API embeds these into the `memo` it returns (THORChain memo format appends `/interval/quantity`). Don't construct the memo yourself — always use the route's `memo` verbatim.

## Important Rules

- `dry: false` requires exactly one provider in the `providers` array.
- If you request a real quote but never send funds, your fulfillment ratio drops and your rate limit decreases.
- Never construct the memo manually — always use the exact `memo` string from the route response.
