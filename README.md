# tc-swap

AI agent skills for [swap.thorchain.org](https://swap.thorchain.org) — a cross-chain cryptocurrency swap interface for THORChain and Mayachain.

## Install

```bash
# skills.sh
npx skills add horizontalsystems/tc-swap

# clawhub
npx clawhub@latest install tc-swap
```

## Setup

Set these environment variables before using the skill:

```
TCSWAP_BASE_URL=https://api.thorchain.org/agent
TCSWAP_AGENT_KEY=<your-agent-key>
```

Register once to get your `TCSWAP_AGENT_KEY`:

```bash
curl -X POST $TCSWAP_BASE_URL/register \
  -H "Content-Type: application/json" \
  -d '{"name": "my-agent"}'
```

## Skills

| Skill | Description |
|---|---|
| `tc-swap` | Get quotes, execute swaps, track status via THORChain and Mayachain |

## Supported Providers

THORChain · Mayachain
