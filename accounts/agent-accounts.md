# Agent Accounts

An agent account is a programmable wallet designed for autonomous use by AI agents or other software. It has its own Base wallet address, USDC balance, and a spending policy that governs every transaction it makes.

## Overview

Agent accounts are owned by a parent personal account. They are created via the API, funded by the operator, and can transact independently within the bounds of their spending policy. No human needs to be present when an agent spends money.

An agent account's identifier is namespaced to its parent: `@username/agent-name`. Every transaction in your feed that originates from an agent shows this namespaced ID, making it clear which agent spent what and when.

## Creating an agent account

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const wallet = await client.wallets.create({
  name: "research-agent",          // becomes @yourhandle/research-agent
  policyId: "pol_conservative",    // required — see Spending Policies
  label: "Research Agent",         // human-readable label for your dashboard
});

console.log(wallet.id);              // "wal_01j..."
console.log(wallet.handle);          // "@yourhandle/research-agent"
console.log(wallet.base_address);    // "0x7fKp..."
console.log(wallet.balance);         // "0.00"
```
{% endtab %}

{% tab title="Python" %}
```python
wallet = client.wallets.create(
    name="research-agent",
    policy_id="pol_conservative",
    label="Research Agent",
)

print(wallet.id)              # "wal_01j..."
print(wallet.handle)          # "@yourhandle/research-agent"
print(wallet.base_address)    # "0x7fKp..."
print(wallet.balance)         # "0.00"
```
{% endtab %}

{% tab title="REST API" %}
```http
POST /v1/wallets
Authorization: Bearer $CLOAK_API_KEY
Content-Type: application/json

{
  "name": "research-agent",
  "policy_id": "pol_conservative",
  "label": "Research Agent"
}
```

```json
{
  "id": "wal_01j...",
  "handle": "@yourhandle/research-agent",
  "base_address": "0x7fKp...",
  "balance": "0.00",
  "currency": "USDC",
  "policy_id": "pol_conservative",
  "status": "active",
  "created_at": "2026-05-13T10:00:00Z"
}
```
{% endtab %}
{% endtabs %}

Every agent account must have a spending policy attached at creation. You cannot create an agent account without one. See [Spending Policies](../policies/README.md).

## Funding an agent account

Fund an agent wallet by sending USDC from your personal account to the agent's handle or Base address.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
// Fund from your personal account
const transfer = await client.transfers.send({
  to: "@yourhandle/research-agent",
  amount: "100.00",
  currency: "USDC",
  memo: "Initial funding",
});
```
{% endtab %}

{% tab title="REST API" %}
```http
POST /v1/transfers
Authorization: Bearer $CLOAK_API_KEY

{
  "to": "@yourhandle/research-agent",
  "amount": "100.00",
  "currency": "USDC",
  "memo": "Initial funding"
}
```
{% endtab %}
{% endtabs %}

You can also fund agent wallets from an external Base wallet by sending USDC to the agent's `base_address` directly.

## Agent-initiated payments

Once funded and given its credentials, an agent can initiate payments autonomously using the API. The agent authenticates with its own API key (scoped to its wallet) and calls the same transfers endpoint.

```typescript
// In your agent's code
const agentClient = new CloakPay({
  apiKey: process.env.AGENT_API_KEY,  // scoped to this agent wallet
});

const tx = await agentClient.transfers.send({
  to: "api.someservice.com",  // x402-compatible endpoint
  amount: "0.025",
  currency: "USDC",
  memo: "API call — search query",
  confidential: true,
});
```

If the transaction would violate the agent's spending policy, the API returns a `403 Policy Violation` error with details on which rule was triggered. The transaction is not submitted to Base.

## Listing agent accounts

```typescript
const wallets = await client.wallets.list();

for (const wallet of wallets.data) {
  console.log(`${wallet.handle}: ${wallet.balance} USDC`);
}
```

## Pausing and closing agent accounts

You can pause an agent account to immediately halt all outbound transactions without deleting the account or its history.

```typescript
await client.wallets.pause("wal_01j...");
await client.wallets.resume("wal_01j...");
await client.wallets.close("wal_01j...");  // drains balance to parent account
```

Closing an agent account withdraws the remaining USDC balance to the parent personal account automatically.

## Agent API keys

Each agent account has its own scoped API key. This key can only operate on that specific wallet. It cannot access your personal account, list other wallets, or perform any action outside the wallet's scope.

Generate and rotate agent API keys from the dashboard or via the API:

```typescript
const key = await client.wallets.createApiKey("wal_01j...");
console.log(key.secret); // shown once — store it securely
```

{% hint style="warning" %}
Agent API key secrets are shown exactly once at creation. CloakPay does not store them. If you lose a key, rotate it immediately.
{% endhint %}
