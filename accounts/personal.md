# Personal Accounts

A personal account is your primary CloakPay account. It holds your USDC balance, maintains your transaction history, and serves as the parent for any agent accounts you create.

## Creating an account

Accounts are created at [usecloakpay.xyz](https://usecloakpay.xyz). You sign up with an email address and a passkey. No seed phrase is shown during onboarding. Your private key is derived from your passkey using standard key derivation (WebAuthn / PBKDF2) and stored only on your device.

If you want to use CloakPay across multiple devices, see [Key Management](key-management.md).

## Your handle

Every personal account gets a handle in the format `@username`. Handles are chosen during signup and are globally unique across CloakPay. You can send and receive payments using a handle instead of a raw Base address.

Handles resolve to the account's current Base wallet address. If you rotate your keys or change your wallet address, your handle continues to work.

## Balance and assets

Personal accounts are USDC-denominated. Your primary balance is displayed in USDC and USD equivalent.

A small ETH reserve is maintained automatically to cover Base transaction fees. You do not need to manage this. When the reserve drops below the threshold required for a transaction, CloakPay tops it up via an internal swap. This happens transparently in the background.

You can also hold ETH directly if you choose to. Swapping between USDC and ETH is supported in-app.

## Sending and receiving

**Sending:** Enter a CloakPay handle, Base address, or request link. Enter an amount in USDC. Add an optional memo. Confidential mode is on by default and can be toggled per transfer.

**Receiving:** Share your handle or QR code. Create a payment request link with a fixed amount and optional memo for invoicing or bill splitting.

**Memos:** Memos are end-to-end encrypted between sender and receiver. They are not visible on-chain to third parties.

## Transaction feed

Your transaction feed shows every transfer in and out of your account. Each entry shows:

- Direction (sent / received)
- Amount in USDC
- Counterparty handle or address
- Memo (if set)
- Timestamp
- Base transaction hash (tap to view on Basescan)
- Confidential indicator

Transfers from your agent accounts appear in a separate feed section, tagged with the agent's name.

## KYC

A lightweight identity verification is required to create a personal account. This is a regulatory requirement under the GENIUS Act (2025) and enables CloakPay to issue and redeem USDC compliantly.

Verification is handled in-app and typically completes in under two minutes. Higher transaction limits require enhanced verification.

| Tier | Monthly limit | Requirement |
|---|---|---|
| Basic | $5,000 | Email + phone |
| Standard | $50,000 | Government ID |
| Enhanced | Unlimited | ID + proof of address |
