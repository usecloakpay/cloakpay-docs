# Selective Disclosure

Selective disclosure lets you prove the contents of a specific transaction to a third party without exposing any other part of your account history.

## When you need it

- Providing transaction evidence to a tax authority or auditor
- Proving payment to a counterparty who disputes receipt
- Demonstrating a specific transfer to legal counsel
- Satisfying travel rule requirements for a specific transaction

## What a disclosure proof contains

A selective disclosure proof is a signed cryptographic document that reveals the following for a single transaction:

- Sender address
- Receiver address
- Amount (decrypted from ciphertext)
- Timestamp
- Base transaction hash
- A cryptographic proof that the above values match the on-chain record

The proof is self-contained and verifiable by anyone with access to the Base RPC. It does not require access to your account or any CloakPay API key to verify.

## Generating a disclosure proof

```http
POST /v1/transfers/tx_01j.../disclose
Authorization: Bearer $CLOAK_API_KEY
```

```json
{
  "transaction_id": "tx_01j...",
  "document": {
    "sender_address": "0x7fKp...",
    "receiver_address": "0x9mHkXxXxXxXxXxXx",
    "amount": "50.00",
    "currency": "USDC",
    "settled_at": "2026-05-13T10:23:41Z",
    "tx_hash": "0x3zKp..."
  },
  "disclosure_signature": "5rJd...",
  "verify_url": "https://usecloakpay.xyz/verify/disc_01j..."
}
```

## Sharing a proof

The `verify_url` links to a public CloakPay page that displays the disclosed transaction details and shows the cryptographic verification status. You can share this URL with anyone. It does not require the recipient to have a CloakPay account.

You can also share the raw `document` and `disclosure_signature` for independent verification without going through CloakPay's servers.

## Verifying a proof independently

The disclosure document can be verified against the on-chain record using any Base RPC node. The verification algorithm is published as an open-source library. Look up the Base transaction by `tx_hash` and confirm the decrypted values in `document` match the on-chain record.

This verification has no dependency on CloakPay's API. It only requires a Base RPC endpoint.

## Audit export

For tax purposes, you can export a complete decrypted transaction history from the dashboard. This export includes every transaction from your account, with amounts decrypted, in CSV or JSON format compatible with major crypto tax tools (Koinly, CoinTracker, etc.).

The audit export is generated client-side using your private key. It is never transmitted to CloakPay's servers in decrypted form.

```http
POST /v1/account/export
Authorization: Bearer $CLOAK_API_KEY
Content-Type: application/json

{
  "from": "2026-01-01",
  "to": "2026-12-31",
  "format": "csv"
}
```

```json
{
  "download_url": "https://..."
}
```
