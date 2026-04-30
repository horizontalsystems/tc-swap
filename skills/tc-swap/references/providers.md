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
