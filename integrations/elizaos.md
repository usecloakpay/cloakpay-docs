# ElizaOS

CloakPay works with ElizaOS through a custom plugin that calls the CloakPay HTTP API. Define the plugin below and pass it to your agent configuration.

## Setup

```typescript
import { createAgent, Plugin, Action } from "@elizaos/core";

const CLOAK_API_KEY = process.env.AGENT_API_KEY;
const BASE_URL = "https://api.usecloakpay.xyz/v1";

const cloakPayPlugin: Plugin = {
  name: "cloakpay",
  actions: [
    {
      name: "SEND_PAYMENT",
      description: "Send USDC from the agent's wallet to a recipient.",
      handler: async (_runtime, _message, state) => {
        const res = await fetch(`${BASE_URL}/transfers`, {
          method: "POST",
          headers: {
            Authorization: `Bearer ${CLOAK_API_KEY}`,
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            to: state.to,
            amount: state.amount,
            currency: "USDC",
            memo: state.memo,
            confidential: true,
          }),
        });
        return res.json();
      },
    },
    {
      name: "CHECK_BALANCE",
      description: "Return the agent's current USDC balance.",
      handler: async () => {
        const res = await fetch(`${BASE_URL}/account`, {
          headers: { Authorization: `Bearer ${CLOAK_API_KEY}` },
        });
        const data = await res.json();
        return { balance: data.balance, currency: data.currency };
      },
    },
    {
      name: "GET_TRANSACTION_HISTORY",
      description: "Return recent transactions from the agent's wallet.",
      handler: async (_runtime, _message, state) => {
        const limit = state.limit ?? 10;
        const res = await fetch(`${BASE_URL}/transfers?limit=${limit}`, {
          headers: { Authorization: `Bearer ${CLOAK_API_KEY}` },
        });
        return res.json();
      },
    },
  ],
};

const agent = createAgent({
  name: "MyAgent",
  plugins: [cloakPayPlugin],
  // ... rest of your agent config
});
```

## Available actions

The plugin registers the following actions:

### `SEND_PAYMENT`

Sends USDC from the agent's wallet to a recipient.

**Natural language examples:**
- "Pay @alice 5 USDC for the data report"
- "Send 0.025 USDC to api.service.com for the search query"

**State fields the agent populates:**

| Field | Type | Description |
|---|---|---|
| `to` | string | Handle or Base address |
| `amount` | string | Amount in USDC |
| `memo` | string | Optional description |

### `CHECK_BALANCE`

Returns the agent's current USDC balance.

**Natural language examples:**
- "What's my current balance?"
- "How much USDC do I have?"

### `GET_TRANSACTION_HISTORY`

Returns recent transactions from the agent's wallet.

**Natural language examples:**
- "Show me my last 10 transactions"
- "What did I spend today?"

### `CREATE_PAYMENT_REQUEST`

Creates a payment request link for a specific amount. Call `POST /v1/transfers/requests` with `amount`, `currency`, and optional `memo`.

**Natural language examples:**
- "Create a $25 payment request for invoice #1042"

## Example: agent that pays for its own API calls

```typescript
const researchAgent = createAgent({
  name: "ResearchAgent",
  plugins: [cloakPayPlugin],
  systemPrompt: `You are a research agent. When you need to use a paid API,
    use SEND_PAYMENT to pay for the service before making the request.
    Always use confidential transfers.`,
});
```
