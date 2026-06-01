# Quickstart

This guide walks you through creating an account and sending your first confidential USDC transfer using the CloakPay HTTP API. It takes about five minutes.

## Prerequisites

- A CloakPay account (sign up at [usecloakpay.xyz](https://usecloakpay.xyz))
- An API key from your dashboard

## 1. Authenticate

All requests authenticate with a Bearer token in the `Authorization` header:

```http
Authorization: Bearer $CLOAK_API_KEY
```

{% hint style="warning" %}
Never hardcode your API key in source code. Use environment variables or a secrets manager.
{% endhint %}

## 2. Check your balance

```http
GET /v1/account
Authorization: Bearer $CLOAK_API_KEY
```

```json
{
  "handle": "@yourhandle",
  "base_address": "0x7fKp...",
  "balance": "100.00",
  "currency": "USDC"
}
```

## 3. Send a confidential transfer

Confidential transfers hide the amount on-chain while keeping sender and receiver addresses visible. This is the default behavior.

```http
POST /v1/transfers
Authorization: Bearer $CLOAK_API_KEY
Content-Type: application/json

{
  "to": "@alice",
  "amount": "25.00",
  "currency": "USDC",
  "memo": "Invoice #1042",
  "confidential": true
}
```

```json
{
  "id": "tx_01j...",
  "status": "confirmed",
  "amount": "25.00",
  "currency": "USDC",
  "confidential": true,
  "signature": "4xKp...",
  "settled_at": "2026-05-13T10:23:41Z"
}
```

The transfer settles on Base mainnet in under 2 seconds. The response is returned once confirmation is received.

## 4. View your transaction feed

```http
GET /v1/transfers?limit=10
Authorization: Bearer $CLOAK_API_KEY
```

```json
{
  "data": [
    {
      "id": "tx_01j...",
      "direction": "outbound",
      "amount": "25.00",
      "currency": "USDC",
      "memo": "Invoice #1042",
      "status": "confirmed"
    }
  ],
  "has_more": false
}
```

## What's next

- [Core Concepts](concepts.md) - Understand privacy, policies, and agent accounts
- [Agent Accounts](../accounts/agent-accounts.md) - Provision wallets for AI agents
- [Confidential Transfers](../privacy/confidential-transfers.md) - Deep dive into the privacy model
- [Spending Policies](../policies/README.md) - Set limits and velocity caps for agents
