# Confidential Transfers

Confidential transfers hide the USDC amount on-chain while keeping sender and receiver addresses public. This is the default for all CloakPay transfers and can be toggled per request.

## How it works

When you initiate a confidential transfer, the following happens:

1. **Client-side proof generation.** Your browser or app generates a zero-knowledge proof that proves your transfer is valid (you have sufficient balance, the amount is positive) without revealing the amount. This computation happens locally. The plaintext amount never leaves your device.

2. **Encrypted submission.** Your app submits an encrypted transfer instruction to Base, along with the ZK proof. The instruction encodes the amount as ciphertext using ElGamal encryption.

3. **On-chain verification.** Base validators verify the ZK proof and confirm the ciphertext is well-formed. They do not learn the amount. The transaction settles.

4. **Receiver decryption.** The receiver's client decrypts the amount using their private key when they view the transaction. No third party can read the amount without the receiver's key.

## Sending a confidential transfer

Confidential mode is enabled by default. You do not need to set it explicitly unless you want to be explicit or disable it.

```http
POST /v1/transfers
Authorization: Bearer $CLOAK_API_KEY
Content-Type: application/json

{
  "to": "@alice",
  "amount": "50.00",
  "currency": "USDC",
  "confidential": true
}
```

To send a non-confidential transfer (amount visible on-chain), set `"confidential": false`.

## What the response looks like

A confirmed confidential transfer response includes the transaction hash but not a human-readable amount field on the on-chain record. The amount is visible to you in your own feed because your client decrypts it locally.

```json
{
  "id": "tx_01j...",
  "status": "confirmed",
  "confidential": true,
  "amount": "50.00",          // decrypted for you locally
  "currency": "USDC",
  "to": "@alice",
  "memo": null,
  "tx_hash": "0x3zKp...",     // Base transaction hash
  "settled_at": "2026-05-13T10:23:41Z"
}
```

If you look up this transaction on Basescan, you will see the sender and receiver addresses and confirmation of the transfer, but the amount field will show as encrypted ciphertext.

## Proof generation performance

ZK proof generation runs in your browser using a compiled WebAssembly module. On modern hardware, proof generation typically takes 200 to 400ms. Combined with Base's settlement time, a confidential transfer typically completes in under 2 seconds total.

On lower-powered devices (older mobile phones), proof generation may take up to 1.5 seconds. CloakPay displays a progress indicator during this phase.

## Confidential transfers for agent accounts

Agent accounts support confidential transfers identically to personal accounts. The proof is generated server-side by the CloakPay API when the agent initiates a transfer. This happens automatically.

```http
POST /v1/transfers
Authorization: Bearer $AGENT_API_KEY
Content-Type: application/json

{
  "to": "api.service.com",
  "amount": "0.05",
  "currency": "USDC"
}
```

## Disabling confidentiality at the account level

You can set confidential mode as a default at the account or wallet level from the dashboard. Individual transfers can still override this setting.

## Limitations

- Confidential transfers require both sender and receiver to have zero-knowledge proofs on Base enabled on their token account. CloakPay handles this setup automatically for all accounts on the platform.
- Transfers to external Base wallets that do not have zero-knowledge proofs on Base enabled will fall back to standard (non-confidential) transfers automatically. The API will indicate this in the response via `"confidential": false`.
- Maximum confidential transfer amount per transaction is $10,000 during the beta period. This limit will be raised post-beta.
