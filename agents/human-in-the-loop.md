# Human-in-the-Loop Controls

Human-in-the-loop (HITL) controls let you define a threshold above which an agent's transactions require explicit human approval before executing. Below the threshold, the agent operates autonomously without interruption.

## How it works

HITL is configured via the `require_co_sign` parameter in a spending policy. When an agent initiates a transfer at or above that amount:

1. The transfer is held in a `pending_co_sign` state
2. A push notification is sent to the parent account holder (mobile app or webhook)
3. The parent reviews the pending transfer in the app or via API
4. The parent approves or rejects
5. If approved, the transfer executes. If rejected or timed out, it is cancelled

The agent's code waits for a result before proceeding. You can configure how long it waits before treating no-response as a rejection.

## Configuring HITL

Set `require_co_sign` when creating or updating a spending policy:

```typescript
const policy = await client.policies.create({
  name: "procurement-agent",
  maxPerTx: "500.00",
  maxPerDay: "2000.00",
  requireCoSign: "100.00",   // anything at or above $100 needs approval
});
```

## The agent's perspective

When an agent initiates a transfer that triggers co-signing, the API does not immediately return a confirmed transaction. Instead it returns a `pending_co_sign` response:

```typescript
const transfer = await agentClient.transfers.send({
  to: "@vendor/billing",
  amount: "150.00",
  currency: "USDC",
  memo: "Monthly data subscription",
});

console.log(transfer.status);  // "pending_co_sign"
console.log(transfer.id);      // "tx_01j..."
```

The agent can poll for the result or use a webhook to be notified when a decision is made.

**Polling:**

```typescript
// Poll until resolved
const resolved = await client.transfers.waitForCoSign("tx_01j...", {
  timeout: 3600,     // seconds — default: 3600 (1 hour)
  interval: 5,       // seconds between polls — default: 5
});

if (resolved.status === "confirmed") {
  console.log("Transfer approved and executed");
} else if (resolved.status === "rejected") {
  console.log("Transfer was rejected by operator");
} else if (resolved.status === "timed_out") {
  console.log("No response within timeout — transfer cancelled");
}
```

**Webhook (recommended for production):**

See [Webhooks](../sdk/rest.md#webhooks) for how to receive `transfer.co_sign_resolved` events.

## The operator's perspective

Pending co-sign requests appear in the CloakPay app under a dedicated "Pending Approvals" section. Each entry shows:

- Agent name and wallet handle
- Transfer amount and currency
- Recipient
- Memo
- Timestamp of the original request
- Time remaining before auto-cancellation

Approvals and rejections can also be made via API:

```typescript
// Approve
await client.transfers.approve("tx_01j...");

// Reject
await client.transfers.reject("tx_01j...", {
  reason: "Unexpected vendor — requires review",  // optional
});
```

## Timeout behaviour

If the operator does not respond within the configured timeout period, the transfer is automatically cancelled. The agent receives a `timed_out` status.

The default timeout is 1 hour. You can configure a different default on the policy:

```typescript
const policy = await client.policies.create({
  name: "with-custom-timeout",
  maxPerTx: "500.00",
  requireCoSign: "100.00",
  coSignTimeoutSeconds: 7200,  // 2 hours
});
```

## Disabling HITL

To remove co-signing requirements, update the policy to remove `require_co_sign`:

```typescript
await client.policies.update("pol_01j...", {
  requireCoSign: null,
});
```

After this update, the agent will execute all transfers within the remaining policy limits autonomously.
