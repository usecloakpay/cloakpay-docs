# Policy Reference

Complete reference for all spending policy parameters.

## Parameters

### `name`

**Type:** string  
**Required:** Yes  

A human-readable name for the policy. Used for display in the dashboard. Must be unique within your account. Immutable after creation.

---

### `max_per_tx`

**Type:** decimal string (e.g., `"5.00"`)  
**Required:** Conditional (required if `max_per_day` is not set)  

Maximum USDC amount allowed in a single transaction. Transactions above this amount are rejected by the on-chain smart contract regardless of remaining daily budget.

---

### `max_per_day`

**Type:** decimal string (e.g., `"50.00"`)  
**Required:** Conditional (required if `max_per_tx` is not set)  

Rolling 24-hour spend ceiling. The window is calculated from the current time minus 24 hours. Transactions that would cause the rolling sum to exceed this value are rejected.

---

### `max_per_month`

**Type:** decimal string (e.g., `"1000.00"`)  
**Required:** No  

Rolling 30-day spend ceiling. Evaluated the same way as `max_per_day`.

---

### `velocity_cap`

**Type:** integer  
**Required:** No  

Maximum number of transactions per hour. Does not affect amounts, only frequency. Useful for preventing runaway loops in agent code.

---

### `allowed_recipients`

**Type:** array of strings  
**Required:** No  

An allowlist of CloakPay handles or Base addresses the agent is permitted to send to. If this array is non-empty, any transfer to a recipient not in the list will be rejected.

Leave empty (or omit) to allow transfers to any recipient, subject to other rules.

```json
"allowed_recipients": [
  "@perplexity",
  "@openai",
  "0x9mHkXxXxXxXxXxXx"
]
```

---

### `blocked_recipients`

**Type:** array of strings  
**Required:** No  

A denylist of CloakPay handles or Base addresses the agent is not permitted to send to. Evaluated after `allowed_recipients`. If a recipient appears in both lists, they are blocked.

---

### `require_co_sign`

**Type:** decimal string (e.g., `"25.00"`)  
**Required:** No  

Transfers equal to or above this amount are held pending and require approval from the parent account before execution. If not set, no co-signing is required regardless of amount.

See [Human-in-the-Loop Controls](../agents/human-in-the-loop.md).

---

### `expires_at`

**Type:** ISO 8601 timestamp (e.g., `"2026-09-01T00:00:00Z"`)  
**Required:** No  

After this timestamp, the wallet's on-chain smart contract rejects all transfer attempts. The wallet can still receive funds after expiry. Set to `null` to remove an existing expiry.

---

### `confidential_only`

**Type:** boolean  
**Default:** `false`  

If `true`, the policy rejects any transfer attempt that does not use confidential mode. Ensures the agent never accidentally broadcasts a payment amount on-chain.

---

## Policy object

```json
{
  "id": "pol_01j...",
  "name": "conservative",
  "max_per_tx": "5.00",
  "max_per_day": "50.00",
  "max_per_month": null,
  "velocity_cap": 20,
  "allowed_recipients": [],
  "blocked_recipients": [],
  "require_co_sign": "25.00",
  "expires_at": null,
  "confidential_only": false,
  "wallet_count": 3,
  "created_at": "2026-05-13T10:00:00Z",
  "updated_at": "2026-05-13T10:00:00Z"
}
```

## Error codes

| Code | Triggered when |
|---|---|
| `policy_max_per_tx` | Transfer amount exceeds `max_per_tx` |
| `policy_max_per_day` | Transfer would exceed rolling 24h ceiling |
| `policy_max_per_month` | Transfer would exceed rolling 30d ceiling |
| `policy_velocity_cap` | Transaction count exceeds `velocity_cap` in the past hour |
| `policy_recipient_not_allowed` | Recipient not in `allowed_recipients` |
| `policy_recipient_blocked` | Recipient is in `blocked_recipients` |
| `policy_co_sign_required` | Amount meets or exceeds `require_co_sign` threshold |
| `policy_expired` | Current time is past `expires_at` |
| `policy_confidential_required` | Transfer attempted without confidential mode and `confidential_only` is true |
