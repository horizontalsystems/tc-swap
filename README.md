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

Set this environment variable before using the skill:

```
TCSWAP_AGENT_KEY=<your-agent-key>
```

Register once to get your `TCSWAP_AGENT_KEY`:

```bash
curl -X POST https://api.thorchain.org/agent/register \
  -H "Content-Type: application/json" \
  -d '{"name": "my-agent"}'
```

## Skills

| Skill | Description |
|---|---|
| `tc-swap` | Get quotes, execute swaps, track status via THORChain and Mayachain |

## Supported Providers

THORChain · Mayachain
