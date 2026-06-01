# REST API

The CloakPay REST API is the primary interface for all CloakPay operations. Use it directly with any HTTP client in any language.

## Base URL

```
https://api.usecloakpay.xyz
```

All endpoints are versioned under `/v1`.

## Authentication

All requests must include an `Authorization` header with a Bearer token:

```http
Authorization: Bearer $CLOAK_API_KEY
```

API keys are created and managed from the CloakPay dashboard. Agent wallet keys are generated via the [Wallets API](#wallets) and are scoped to a single wallet.

## Request format

- All request bodies must be JSON
- Set `Content-Type: application/json` on all `POST` and `PATCH` requests
- All amounts are decimal strings (e.g., `"50.00"`)
- All timestamps are ISO 8601 in UTC

## Response format

All responses return JSON. Successful responses use 2xx status codes. Errors use 4xx or 5xx status codes with a structured error body.

**Success:**
```json
{
  "id": "...",
  "status": "...",
  "..."
}
```

**Error:**
```json
{
  "error": {
    "code": "policy_max_per_tx",
    "message": "Transfer amount exceeds max_per_tx limit",
    "details": {
      "rule": "max_per_tx",
      "limit": "5.00",
      "attempted": "50.00"
    }
  }
}
```

## Pagination

List endpoints use cursor-based pagination:

```http
GET /v1/transfers?limit=20&cursor=cur_01j...
```

Responses include a `next_cursor` field when more results are available:

```json
{
  "data": [...],
  "has_more": true,
  "next_cursor": "cur_01j..."
}
```

---

## Account

### Get account

```http
GET /v1/account
```

```json
{
  "handle": "@yourhandle",
  "base_address": "0x7fKp...",
  "balance": "100.00",
  "currency": "USDC",
  "kyc_tier": "standard",
  "created_at": "2026-01-01T00:00:00Z"
}
```

---

## Transfers

### Send a transfer

```http
POST /v1/transfers
```

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `to` | string | Yes | Handle, Base address, or payment request URL |
| `amount` | decimal string | Yes | Amount in USDC |
| `currency` | string | Yes | Must be `"USDC"` |
| `memo` | string | No | End-to-end encrypted memo |
| `confidential` | boolean | No | Default: `true` |

**Response:**

```json
{
  "id": "tx_01j...",
  "status": "confirmed",
  "amount": "50.00",
  "currency": "USDC",
  "to": "@alice",
  "memo": "Invoice #1042",
  "confidential": true,
  "tx_hash": "0x3zKp...",
  "settled_at": "2026-05-13T10:23:41Z"
}
```

### Get a transfer

```http
GET /v1/transfers/:id
```

### List transfers

```http
GET /v1/transfers?limit=20&direction=outbound&cursor=cur_01j...
```

### Create a payment request

```http
POST /v1/transfers/requests
```

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `amount` | decimal string | No | Pre-filled amount. If omitted, payer chooses |
| `currency` | string | No | Default: `"USDC"` |
| `memo` | string | No | Memo shown to payer |
| `expires_in` | integer | No | Seconds until expiry. Default: 86400 |

### Generate a disclosure proof

```http
POST /v1/transfers/:id/disclose
```

---

## Wallets

### Create a wallet

```http
POST /v1/wallets
```

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | URL-safe name. Becomes `@yourhandle/name` |
| `policy_id` | string | Yes | ID of an existing policy |
| `label` | string | No | Human-readable label for the dashboard |

**Response includes `private_key` exactly once. Store it securely.**

### Get a wallet

```http
GET /v1/wallets/:id
```

### List wallets

```http
GET /v1/wallets
```

### Pause / resume / close

```http
POST /v1/wallets/:id/pause
POST /v1/wallets/:id/resume
POST /v1/wallets/:id/close
```

Closing a wallet sweeps the USDC balance to the parent account.

### Create a scoped API key

```http
POST /v1/wallets/:id/keys
```

**Response:**
```json
{
  "id": "key_01j...",
  "secret": "hp_live_...",    // shown once only
  "wallet_id": "wal_01j...",
  "created_at": "2026-05-13T10:00:00Z"
}
```

---

## Policies

### Create a policy

```http
POST /v1/policies
```

See [Policy Reference](../policies/reference.md) for all parameters.

### Get a policy

```http
GET /v1/policies/:id
```

### List policies

```http
GET /v1/policies
```

### Update a policy

```http
PATCH /v1/policies/:id
```

### Delete a policy

```http
DELETE /v1/policies/:id
```

Returns `409 Conflict` if wallets are still using the policy.

---

## Approvals

### List pending approvals

```http
GET /v1/approvals?status=pending
```

### Approve a transfer

```http
POST /v1/approvals/:transfer_id/approve
```

### Reject a transfer

```http
POST /v1/approvals/:transfer_id/reject
```

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `reason` | string | No | Reason surfaced to the agent |

---

## Webhooks

### Create a webhook endpoint

```http
POST /v1/webhooks
```

**Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `url` | string | Yes | HTTPS URL to deliver events to |
| `events` | string array | Yes | List of event types to subscribe to |
| `secret` | string | No | Used to sign payloads. Generated if not provided |

**Available event types:**

| Event | Triggered when |
|---|---|
| `transfer.confirmed` | A transfer settles successfully |
| `transfer.failed` | A transfer fails |
| `transfer.co_sign_requested` | A transfer needs co-sign approval |
| `transfer.co_sign_resolved` | A pending approval is approved or rejected |
| `wallet.low_balance` | A wallet's USDC balance falls below a threshold |
| `wallet.policy_violation` | A transfer is blocked by the spending policy |

**Payload verification:**

Every webhook request includes a `Cloakpay-Signature` header containing an HMAC-SHA256 signature of the raw request body, signed with your webhook secret:

```
Cloakpay-Signature: t=1715595821,v1=a3f9...
```

Verify the signature before processing the payload.

---

## Error codes

| Code | HTTP status | Description |
|---|---|---|
| `unauthorized` | 401 | Invalid or missing API key |
| `forbidden` | 403 | Key does not have permission for this resource |
| `not_found` | 404 | Resource does not exist |
| `insufficient_balance` | 422 | Sender does not have enough USDC |
| `policy_max_per_tx` | 422 | Transfer exceeds per-transaction limit |
| `policy_max_per_day` | 422 | Transfer would exceed daily limit |
| `policy_max_per_month` | 422 | Transfer would exceed monthly limit |
| `policy_velocity_cap` | 422 | Velocity cap reached for this hour |
| `policy_recipient_not_allowed` | 422 | Recipient not in allowlist |
| `policy_recipient_blocked` | 422 | Recipient is in blocklist |
| `policy_co_sign_required` | 202 | Transfer held pending co-sign approval |
| `policy_expired` | 422 | Wallet's policy has passed its expiry date |
| `policy_confidential_required` | 422 | Non-confidential transfer rejected by policy |
| `wallet_paused` | 422 | Wallet is paused and cannot transact |
| `wallet_closed` | 410 | Wallet has been closed |
| `rate_limited` | 429 | Too many requests |
| `internal_error` | 500 | Unexpected server error |

## Rate limits

API keys are rate-limited to 100 requests per second by default. Agent wallet keys inherit this limit unless you contact support to raise it.

Rate limit headers are included in every response:

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 97
X-RateLimit-Reset: 1715595900
```
