# Spending Policies

A spending policy is a set of rules that governs what an agent account can spend, how fast, and with whom. Every agent account must have a policy. You cannot create an agent wallet without one.

## Why policies exist at the program level

Most financial controls live in application code. If your application sends a payment request, the payment goes through. The rules are checked by your code before the API call, not by the payment system itself. This creates a problem: if the application code has a bug, or if someone calls the API directly, the rules can be bypassed.

CloakPay policies are enforced by an on-chain Base smart contract. When an agent attempts a transfer, the instruction is evaluated by the smart contract before execution. If the transfer violates the policy, the smart contract rejects the instruction. The transaction is never submitted to the validator network.

This means policy enforcement is not optional, cannot be bypassed by calling the API differently, and takes effect in under 2 seconds when updated.

## Policy parameters

| Parameter | Type | Description |
|---|---|---|
| `max_per_tx` | decimal string | Maximum USDC per single transaction |
| `max_per_day` | decimal string | Rolling 24-hour spend ceiling |
| `max_per_month` | decimal string | Rolling 30-day spend ceiling |
| `velocity_cap` | integer | Maximum transactions per hour |
| `allowed_recipients` | string array | Allowlist of handles or Base addresses. Empty means all are allowed |
| `blocked_recipients` | string array | Denylist of handles or Base addresses |
| `require_co_sign` | decimal string | Amount above which operator co-signature is required |
| `expires_at` | ISO 8601 timestamp | Policy expiry. After this date, the wallet cannot transact |
| `confidential_only` | boolean | If true, rejects any non-confidential transfer attempt |

## Creating a policy

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const policy = await client.policies.create({
  name: "conservative",
  maxPerTx: "5.00",
  maxPerDay: "50.00",
  velocityCap: 20,
  requireCoSign: "25.00",
});

console.log(policy.id); // "pol_01j..."
```
{% endtab %}

{% tab title="Python" %}
```python
policy = client.policies.create(
    name="conservative",
    max_per_tx="5.00",
    max_per_day="50.00",
    velocity_cap=20,
    require_co_sign="25.00",
)

print(policy.id)  # "pol_01j..."
```
{% endtab %}

{% tab title="REST API" %}
```http
POST /v1/policies
Authorization: Bearer $CLOAK_API_KEY
Content-Type: application/json

{
  "name": "conservative",
  "max_per_tx": "5.00",
  "max_per_day": "50.00",
  "velocity_cap": 20,
  "require_co_sign": "25.00"
}
```

```json
{
  "id": "pol_01j...",
  "name": "conservative",
  "max_per_tx": "5.00",
  "max_per_day": "50.00",
  "velocity_cap": 20,
  "require_co_sign": "25.00",
  "created_at": "2026-05-13T10:00:00Z"
}
```
{% endtab %}
{% endtabs %}

Policies can be reused across multiple wallets. If you update a policy, the change applies to all wallets using that policy within one Base block.

## Attaching a policy to a wallet

A policy is specified at wallet creation time and cannot be removed, only updated or replaced.

```typescript
const wallet = await client.wallets.create({
  name: "api-agent",
  policyId: policy.id,
  label: "API Agent",
});
```

## Updating a policy

Policy changes are applied on-chain and take effect in under 2 seconds.

```typescript
await client.policies.update("pol_01j...", {
  maxPerDay: "100.00",  // increase the daily limit
});
```

All wallets using this policy will immediately be subject to the new rules.

## Policy violation errors

When an agent's transfer is rejected by a policy, the API returns a structured error:

```json
{
  "error": {
    "code": "policy_violation",
    "message": "Transfer amount exceeds max_per_tx limit",
    "details": {
      "rule": "max_per_tx",
      "limit": "5.00",
      "attempted": "12.50",
      "currency": "USDC"
    }
  }
}
```

## Built-in policy templates

CloakPay provides three built-in policy templates as a starting point. You can use them directly by referencing their IDs, or clone them and customise.

| Template ID | Description |
|---|---|
| `pol_restrictive` | $1 per tx, $10 per day, 5 tx/hr, no co-sign |
| `pol_conservative` | $5 per tx, $50 per day, 20 tx/hr, co-sign above $25 |
| `pol_standard` | $25 per tx, $250 per day, 50 tx/hr, co-sign above $100 |

```typescript
// Use a built-in template
const wallet = await client.wallets.create({
  name: "my-agent",
  policyId: "pol_conservative",  // built-in template
});
```
