# Providers, Chains & Asset Format

## Asset ID Format

```
CHAIN.SYMBOL
CHAIN.SYMBOL-CONTRACT_ADDRESS
```

**Examples:**

| Asset | ID |
|---|---|
| Bitcoin | `BTC.BTC` |
| Ether | `ETH.ETH` |
| USDC on Ethereum | `ETH.USDC-0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` |
| BNB native | `BSC.BNB` |
| RUNE | `THOR.RUNE` |
| CACAO | `MAYA.CACAO` |

### Secured Assets (THORCHAIN only)

THORChain Secured Assets use a **dash** instead of a dot — `CHAIN-SYMBOL[-CONTRACT]`. They're 1:1-backed claims on L1, held in THORChain's `x/bank` module, denominated in 8 decimals regardless of L1 native decimals.

| Asset | ID |
|---|---|
| Secured ETH | `ETH-ETH` |
| Secured BTC | `BTC-BTC` |
| Secured USDC on Ethereum | `ETH-USDC-0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` |

**Constraints when quoting:**
- **L1 → Secured:** `destinationAddress` must be a `thor1…` address (the secured-asset balance lands on THORChain).
- **Secured → L1:** `sourceAddress` must be the `thor1…` address holding the secured balance. The deposit is a `MsgDeposit` on THORChain — **the server does not build this transaction; the agent must construct and sign the `MsgDeposit` with the returned `memo`**. Inbound fee is reported as `0.02 RUNE` (a fixed gas approximation).
- **Secured → Secured:** both legs on THORChain; same `MsgDeposit` constraint.

Trade Assets (`~`), synthetics (`/`), and derived assets (`THOR.X`) are **not** supported.

## Providers

Both providers are decentralized DEXes — no AML blocking, no KYC.

| Provider | Supported Chains |
|---|---|
| THORCHAIN | BTC, BCH, LTC, DOGE, ETH, AVAX, BSC, BASE, GAIA, TRON, XRP, THOR |
| MAYACHAIN | BTC, ETH, DASH, ZEC, KUJI, ARB, RADIX, THOR, MAYA |

## Token Lists

```
GET https://api.thorchain.org/agent/v1/tokens?provider=THORCHAIN
X-Agent-Key: $TCSWAP_AGENT_KEY
```

Returns supported tokens for the given provider.

## List Providers

```
GET https://api.thorchain.org/agent/v1/providers
X-Agent-Key: $TCSWAP_AGENT_KEY
```

## Rate Limiting

- Starts at **15 req/hour** for new agents
- Scales up automatically with fulfillment ratio (up to **100 req/hour**)
- Agents with ≥50% fulfillment ratio receive a **3× bonus** (up to **200 req/hour**)
- Creating real orders (`dry: false`) and then tracking completed swaps improves your ratio
- After 3 suspensions an agent is permanently banned
