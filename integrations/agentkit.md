# Coinbase AgentKit

CloakPay works with AgentKit through a custom `ActionProvider` that calls the CloakPay HTTP API. Implement the provider below and pass it to `AgentKit`.

## Setup

```typescript
import { AgentKit, ActionProvider, Action } from "@coinbase/agentkit";
import { ChatOpenAI } from "@langchain/openai";

class CloakPayActionProvider extends ActionProvider {
  private apiKey: string;

  constructor({ apiKey }: { apiKey: string }) {
    super();
    this.apiKey = apiKey;
  }

  getActions(): Action[] {
    return [
      {
        name: "cloakpay_send_payment",
        description: "Send USDC to a recipient via CloakPay.",
        schema: {
          type: "object",
          properties: {
            to: { type: "string", description: "@handle or Base address" },
            amount: { type: "string", description: "Decimal amount e.g. '5.00'" },
            memo: { type: "string" },
            confidential: { type: "boolean", default: true },
          },
          required: ["to", "amount"],
        },
        invoke: async ({ to, amount, memo, confidential = true }) => {
          const res = await fetch("https://api.usecloakpay.xyz/v1/transfers", {
            method: "POST",
            headers: {
              Authorization: `Bearer ${this.apiKey}`,
              "Content-Type": "application/json",
            },
            body: JSON.stringify({ to, amount, currency: "USDC", memo, confidential }),
          });
          return res.json();
        },
      },
      {
        name: "cloakpay_get_balance",
        description: "Get the agent's current USDC balance.",
        schema: { type: "object", properties: {} },
        invoke: async () => {
          const res = await fetch("https://api.usecloakpay.xyz/v1/account", {
            headers: { Authorization: `Bearer ${this.apiKey}` },
          });
          const data = await res.json();
          return { balance: data.balance, currency: data.currency };
        },
      },
      {
        name: "cloakpay_list_transactions",
        description: "Get recent transaction history.",
        schema: {
          type: "object",
          properties: {
            limit: { type: "number", default: 10 },
          },
        },
        invoke: async ({ limit = 10 }) => {
          const res = await fetch(
            `https://api.usecloakpay.xyz/v1/transfers?limit=${limit}`,
            { headers: { Authorization: `Bearer ${this.apiKey}` } },
          );
          return res.json();
        },
      },
    ];
  }
}

const agentKit = new AgentKit({
  actionProviders: [
    new CloakPayActionProvider({
      apiKey: process.env.AGENT_API_KEY,
    }),
  ],
});
```

## Available actions

### `cloakpay_send_payment`

Send USDC to a recipient.

```typescript
// Action schema — the LLM populates these fields
{
  to: string;         // "@handle" or Base address
  amount: string;     // decimal string e.g. "5.00"
  memo?: string;
  confidential?: boolean;  // default: true
}
```

### `cloakpay_get_balance`

Get the agent's current USDC balance.

### `cloakpay_list_transactions`

Get recent transaction history.

### `cloakpay_create_request`

Create a payment request URL for a specific amount. Call `POST /v1/transfers/requests` with `amount`, `currency`, and optional `memo`.

## Full example

```typescript
import { AgentKit, createReActAgent } from "@coinbase/agentkit";
import { ChatOpenAI } from "@langchain/openai";

const agentKit = new AgentKit({
  actionProviders: [
    new CloakPayActionProvider({
      apiKey: process.env.AGENT_API_KEY,
    }),
  ],
});

const llm = new ChatOpenAI({ model: "gpt-4o" });

const agent = createReActAgent(llm, agentKit.getActions());

const response = await agent.invoke({
  messages: [
    {
      role: "user",
      content: "Check my balance and then pay @perplexity 0.05 USDC for a search",
    },
  ],
});
```
