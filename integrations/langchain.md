# LangChain

CloakPay works with LangChain agents through custom `Tool` implementations that call the CloakPay HTTP API. Define the tools below and add them to any LangChain agent.

## TypeScript / JavaScript

```typescript
import { Tool } from "langchain/tools";
import { createReActAgent, AgentExecutor } from "langchain/agents";
import { ChatOpenAI } from "@langchain/openai";

class CloakPayPaymentTool extends Tool {
  name = "cloakpay_send_payment";
  description =
    "Send USDC to a CloakPay handle or Base address. Input should be a JSON object with fields: to (string), amount (string), memo (optional string). Returns the transaction ID and status.";
  private apiKey: string;

  constructor({ apiKey }: { apiKey: string }) {
    super();
    this.apiKey = apiKey;
  }

  async _call(input: string): Promise<string> {
    const { to, amount, memo } = JSON.parse(input);
    const res = await fetch("https://api.usecloakpay.xyz/v1/transfers", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${this.apiKey}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ to, amount, currency: "USDC", memo, confidential: true }),
    });
    const data = await res.json();
    return JSON.stringify({ id: data.id, status: data.status });
  }
}

class CloakPayBalanceTool extends Tool {
  name = "cloakpay_get_balance";
  description =
    "Get the current USDC balance of the agent's CloakPay wallet. Takes no input. Returns the balance as a decimal string.";
  private apiKey: string;

  constructor({ apiKey }: { apiKey: string }) {
    super();
    this.apiKey = apiKey;
  }

  async _call(): Promise<string> {
    const res = await fetch("https://api.usecloakpay.xyz/v1/account", {
      headers: { Authorization: `Bearer ${this.apiKey}` },
    });
    const data = await res.json();
    return data.balance;
  }
}

const tools = [
  new CloakPayPaymentTool({ apiKey: process.env.AGENT_API_KEY }),
  new CloakPayBalanceTool({ apiKey: process.env.AGENT_API_KEY }),
];

const llm = new ChatOpenAI({ model: "gpt-4o", temperature: 0 });

const agent = createReActAgent({ llm, tools });
const executor = new AgentExecutor({ agent, tools, verbose: true });

const result = await executor.invoke({
  input: "Check my balance, then pay @datavendor 2.50 USDC for the market data feed",
});
```

## Python

```python
import os
import json
import requests
from langchain.tools import BaseTool
from langchain.agents import initialize_agent, AgentType
from langchain_openai import ChatOpenAI

class CloakPayPaymentTool(BaseTool):
    name = "cloakpay_send_payment"
    description = (
        "Send USDC to a CloakPay handle or Base address. "
        "Input should be a JSON string with fields: to, amount, memo (optional)."
    )
    api_key: str

    def _run(self, input_str: str) -> str:
        payload = json.loads(input_str)
        resp = requests.post(
            "https://api.usecloakpay.xyz/v1/transfers",
            headers={
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json",
            },
            json={
                "to": payload["to"],
                "amount": payload["amount"],
                "currency": "USDC",
                "memo": payload.get("memo"),
                "confidential": True,
            },
        )
        data = resp.json()
        return json.dumps({"id": data["id"], "status": data["status"]})

class CloakPayBalanceTool(BaseTool):
    name = "cloakpay_get_balance"
    description = "Get the current USDC balance of the agent's CloakPay wallet. Takes no input."
    api_key: str

    def _run(self, _: str = "") -> str:
        resp = requests.get(
            "https://api.usecloakpay.xyz/v1/account",
            headers={"Authorization": f"Bearer {self.api_key}"},
        )
        return resp.json()["balance"]

tools = [
    CloakPayPaymentTool(api_key=os.environ["AGENT_API_KEY"]),
    CloakPayBalanceTool(api_key=os.environ["AGENT_API_KEY"]),
]

llm = ChatOpenAI(model="gpt-4o", temperature=0)
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.OPENAI_FUNCTIONS,
    verbose=True,
)

result = agent.invoke("Pay @perplexity 0.025 USDC for a search query")
```

## Available tools

### `cloakpay_send_payment`

**Description given to the LLM:**
> Send USDC to a CloakPay handle or Base address. Input should be a JSON object with fields: `to` (string), `amount` (string), `memo` (optional string). Returns the transaction ID and status.

**Input schema:**
```json
{
  "to": "@alice",
  "amount": "5.00",
  "memo": "optional memo"
}
```

### `cloakpay_get_balance`

**Description given to the LLM:**
> Get the current USDC balance of the agent's CloakPay wallet. Takes no input. Returns the balance as a decimal string.

### `cloakpay_list_transactions`

**Description given to the LLM:**
> Get a list of recent transactions from the agent's CloakPay wallet. Input should be a JSON object with optional `limit` (integer, default 10) and `direction` ("inbound" | "outbound" | null).

Implement this tool by calling:

```http
GET /v1/transfers?limit=10
Authorization: Bearer $AGENT_API_KEY
```

## Confidential mode

All payment requests default to `"confidential": true`. To send non-confidential transfers, set `"confidential": false` in the request body passed to `POST /v1/transfers`.
