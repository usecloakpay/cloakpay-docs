# Creating Policies

This page covers policy creation in detail, including how to model common use cases with the available parameters.

## Basic structure

A policy is a named set of rules. Once created, you reference it by ID when provisioning wallets. Policies are reusable across multiple wallets.

```typescript
const policy = await client.policies.create({
  name: "api-agent-policy",
  maxPerTx: "5.00",
  maxPerDay: "100.00",
  maxPerMonth: "1000.00",
  velocityCap: 30,
});
```

At minimum, a valid policy must define either `maxPerTx` or `maxPerDay`. All other parameters are optional.

## Modelling common use cases

### API-paying agent

An agent that pays for external API calls should have low per-transaction limits and a reasonable daily ceiling.

```typescript
const policy = await client.policies.create({
  name: "api-paying-agent",
  maxPerTx: "1.00",       // individual API calls are cheap
  maxPerDay: "50.00",     // reasonable daily cap
  velocityCap: 100,       // high velocity for automated usage
});
```

### Research agent with recipient allowlist

Restrict an agent to paying only known services.

```typescript
const policy = await client.policies.create({
  name: "research-agent",
  maxPerTx: "5.00",
  maxPerDay: "100.00",
  allowedRecipients: [
    "@perplexity",
    "@openai",
    "0x9mHkXxXxXxXxXxXx",  // raw Base address
  ],
});
```

When `allowedRecipients` is set, the agent can only send to addresses in that list. Attempts to send to any other address are rejected.

### High-value operations with co-signing

For agents that occasionally need to make larger payments but should not do so unilaterally.

```typescript
const policy = await client.policies.create({
  name: "procurement-agent",
  maxPerTx: "100.00",
  maxPerDay: "500.00",
  requireCoSign: "50.00",  // anything above $50 needs human approval
});
```

When an agent initiates a transfer above the `requireCoSign` threshold, the transaction is held pending and a push notification is sent to the parent account. See [Human-in-the-Loop Controls](../agents/human-in-the-loop.md).

### Time-limited agent

For a temporary agent with a hard cutoff.

```typescript
const policy = await client.policies.create({
  name: "event-agent",
  maxPerTx: "10.00",
  maxPerDay: "200.00",
  expiresAt: "2026-09-01T00:00:00Z",  // agent cannot transact after this date
});
```

After the expiry timestamp, the wallet's on-chain smart contract rejects all transfer attempts. You will need to update the policy to extend or remove the expiry.

### Confidential-only policy

Ensure an agent never makes a non-confidential transfer.

```typescript
const policy = await client.policies.create({
  name: "confidential-agent",
  maxPerTx: "5.00",
  maxPerDay: "100.00",
  confidentialOnly: true,
});
```

## Cloning a template

Clone a built-in template and customise it:

```typescript
const policy = await client.policies.clone("pol_conservative", {
  name: "my-conservative",
  maxPerDay: "200.00",  // override just this field
});
```

## Updating a policy

Updates are applied on-chain and take effect within one Base block (~2 seconds). All wallets using the policy are immediately subject to the new rules.

```typescript
await client.policies.update("pol_01j...", {
  maxPerTx: "10.00",
  velocityCap: 50,
});
```

You can update any field except the policy's `name` (which is immutable). If you need to rename a policy, clone it under the new name.

## Deleting a policy

A policy cannot be deleted while wallets are using it. Reassign or close all wallets that reference the policy first, then delete it.

```typescript
await client.policies.delete("pol_01j...");
// throws PolicyInUseError if any wallets still reference this policy
```
