# Key Management

CloakPay is non-custodial. Your private keys are yours. This page explains how they are generated, stored, and how you can manage them.

## Personal account keys

When you create a personal account, your private key is derived from your device passkey using WebAuthn and PBKDF2 key derivation. The process works as follows:

1. Your device authenticates with your passkey (biometric, PIN, or security key)
2. The passkey assertion is used as input material for PBKDF2 key derivation
3. The derived key becomes your EVM private key
4. The key is stored in your device's secure enclave or keystore
5. CloakPay only ever sees signed transactions, never the key itself

This means your key is tied to your device credential by default. It is not uploaded to CloakPay servers at any point.

## Exporting your key

You can export your EVM private key at any time from the security settings in the app. Once exported, you can import it into any EVM-compatible wallet (MetaMask, Rabby, CLI, etc.).

{% hint style="warning" %}
Your exported private key grants full access to your wallet. Store it in a password manager or hardware security device. Anyone with this key can drain your account.
{% endhint %}

Exported keys are displayed as a hex-encoded string, compatible with EVM wallet tools and most wallet applications:

```
cast wallet import
# or import the hex string directly into MetaMask / Rabby
```

## Multi-device access

Because your key is derived from your device passkey, your account is initially tied to a single device. To access CloakPay on a second device, you have two options:

**Option 1: Import your exported key.** Export your key from your primary device and import it on the second device. Both devices will share the same wallet address.

**Option 2: Use a hardware wallet.** Connect a Ledger or other hardware wallet to the CloakPay web app. Sign transactions with the hardware device rather than a software passkey.

## Key rotation

Key rotation replaces your current EVM keypair with a new one while preserving your handle, account history, and settings. Your new address is linked to your handle automatically.

To rotate your key:

1. Go to Settings > Security > Key Management
2. Select "Rotate Key"
3. Authenticate with your current passkey
4. Confirm the rotation

After rotation, your old address no longer controls the account. Any USDC at the old address is swept to the new one as part of the rotation transaction.

{% hint style="info" %}
Key rotation is an on-chain operation and costs a small Base transaction fee. It is broadcast to the network and the old-to-new address link is recorded in the CloakPay handle registry.
{% endhint %}

## Agent account keys

Agent account keys are generated server-side at the time of wallet creation and returned in the API response. They are generated using the `generatePrivateKey()` method and are not derived from any human credential.

The private key is included in the `create` response exactly once:

```json
{
  "id": "wal_01j...",
  "base_address": "0x9mHkXxXxXxXxXxXx",
  "private_key": "0x5J7Q...",  // hex — shown once only
  "balance": "0.00"
}
```

After this initial response, CloakPay does not store the private key. You are responsible for storing it securely. If you lose it, you cannot recover it through CloakPay.

**If you lose an agent key:** Rotate it from the dashboard. The new key will control the same wallet address. Your old key will be invalidated.

**Recommended storage:** Secrets manager (AWS Secrets Manager, HashiCorp Vault, Doppler), not environment variables in plaintext config files.

## Recovery

If you lose access to your personal account passkey:

1. CloakPay does not offer custodial recovery. We cannot restore access to an account whose key we have never seen.
2. If you have a previously exported private key, you can import it to regain access.
3. If you have no backup and lose your passkey, access to the wallet is lost permanently.

This is the trade-off of a non-custodial model. Your money cannot be seized, frozen, or accessed by CloakPay. It also cannot be recovered by CloakPay if you lose your key.

**We strongly recommend exporting your private key and storing it in a password manager within 24 hours of creating your account.**
