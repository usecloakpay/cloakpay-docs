# OpenAI Agents

The OpenAI Assistants API supports custom function tools. CloakPay integrates by exposing its HTTP API as function definitions that the assistant can call.

## Setup

Define the tool functions and their JSON schemas, then wire up a dispatch loop to handle tool calls by making HTTP requests to CloakPay.

```python
import os
import json
import requests
from openai import OpenAI

CLOAK_API_KEY = os.environ["AGENT_API_KEY"]
BASE_URL = "https://api.usecloakpay.xyz/v1"
HEADERS = {"Authorization": f"Bearer {CLOAK_API_KEY}", "Content-Type": "application/json"}

def cloakpay_send_payment(to: str, amount: str, memo: str = None) -> dict:
    resp = requests.post(
        f"{BASE_URL}/transfers",
        headers=HEADERS,
        json={"to": to, "amount": amount, "currency": "USDC", "memo": memo, "confidential": True},
    )
    return resp.json()

def cloakpay_get_balance() -> dict:
    resp = requests.get(f"{BASE_URL}/account", headers=HEADERS)
    data = resp.json()
    return {"balance": data["balance"], "currency": data["currency"]}

def cloakpay_list_transactions(limit: int = 10) -> dict:
    resp = requests.get(f"{BASE_URL}/transfers?limit={limit}", headers=HEADERS)
    return resp.json()

TOOL_HANDLERS = {
    "cloakpay_send_payment": cloakpay_send_payment,
    "cloakpay_get_balance": cloakpay_get_balance,
    "cloakpay_list_transactions": cloakpay_list_transactions,
}
```

## Tool definitions

Pass these function definitions to the OpenAI assistant:

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "cloakpay_send_payment",
            "description": "Send USDC to a recipient via CloakPay.",
            "parameters": {
                "type": "object",
                "properties": {
                    "to": {"type": "string", "description": "Handle (@alice) or Base address"},
                    "amount": {"type": "string", "description": "Decimal amount e.g. '5.00'"},
                    "memo": {"type": "string", "description": "Optional memo"},
                },
                "required": ["to", "amount"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "cloakpay_get_balance",
            "description": "Get the current USDC balance of the agent's CloakPay wallet.",
            "parameters": {"type": "object", "properties": {}},
        },
    },
    {
        "type": "function",
        "function": {
            "name": "cloakpay_list_transactions",
            "description": "Get recent transaction history.",
            "parameters": {
                "type": "object",
                "properties": {
                    "limit": {"type": "integer", "default": 10},
                },
            },
        },
    },
]
```

## Full example with Assistants API

```python
import json
import os
from openai import OpenAI

openai_client = OpenAI()

# Create assistant with CloakPay tools
assistant = openai_client.beta.assistants.create(
    name="Payment Agent",
    instructions="You are a helpful agent that can make payments on behalf of the user.",
    tools=tools,
    model="gpt-4o",
)

# Create a thread and run
thread = openai_client.beta.threads.create()
openai_client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="Pay @perplexity 0.05 USDC for a search and confirm once done",
)

run = openai_client.beta.threads.runs.create_and_poll(
    thread_id=thread.id,
    assistant_id=assistant.id,
)

# Handle tool calls
while run.status == "requires_action":
    tool_calls = run.required_action.submit_tool_outputs.tool_calls
    tool_outputs = []

    for tool_call in tool_calls:
        fn_name = tool_call.function.name
        fn_args = json.loads(tool_call.function.arguments)
        result = TOOL_HANDLERS[fn_name](**fn_args)
        tool_outputs.append({
            "tool_call_id": tool_call.id,
            "output": json.dumps(result),
        })

    run = openai_client.beta.threads.runs.submit_tool_outputs_and_poll(
        thread_id=thread.id,
        run_id=run.id,
        tool_outputs=tool_outputs,
    )

print(run.status)  # "completed"
```

## Available tools

| Function name | Description |
|---|---|
| `cloakpay_send_payment` | Send USDC to a handle or address |
| `cloakpay_get_balance` | Get current wallet balance |
| `cloakpay_list_transactions` | Get recent transaction history |
| `cloakpay_create_payment_request` | Create a payment request link (`POST /v1/transfers/requests`) |
| `cloakpay_get_transaction` | Look up a specific transaction by ID (`GET /v1/transfers/:id`) |
