# Integrations

CloakPay integrates with the major AI agent frameworks. Each integration shows how to call the CloakPay HTTP API from within that framework's tool or plugin pattern.

## Available integrations

| Framework | Status |
|---|---|
| [ElizaOS](elizaos.md) | Beta |
| [Coinbase AgentKit](agentkit.md) | Beta |
| [LangChain](langchain.md) | Beta |
| [OpenAI Agents](openai-agents.md) | Beta |

## How integrations work

Each integration wraps the CloakPay REST API in the idiomatic pattern of its framework. For example:

- In **ElizaOS**, CloakPay is a plugin that registers `sendPayment`, `checkBalance`, and `getTransactionHistory` actions
- In **LangChain**, CloakPay is a `Tool` with a description that the LLM uses to decide when to invoke it
- In **AgentKit**, CloakPay is an `ActionProvider` that contributes actions to the agent's action set

In all cases, the agent authenticates with a scoped API key tied to its agent wallet. Transactions are bounded by the wallet's spending policy regardless of what the framework or LLM instructs.

## Using a framework not listed here

If your framework is not listed, call the [REST API](../sdk/rest.md) directly. Any HTTP client works. The pattern is always the same: set the `Authorization: Bearer $CLOAK_API_KEY` header and send JSON requests to `https://api.usecloakpay.xyz/v1`.
