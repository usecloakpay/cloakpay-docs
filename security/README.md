# Security and Compliance

This page covers CloakPay's security model, how user funds are protected, and the compliance framework the product operates under.

## Non-custodial architecture

CloakPay does not hold user funds. Every USDC balance in the system sits in a Base wallet controlled by the account holder's private key. CloakPay's servers have no access to these keys and cannot initiate transactions on your behalf.

For personal accounts, keys are derived from your device passkey and stored in your device's secure enclave or keystore. For agent accounts, keys are generated during wallet creation and returned to the operator exactly once. CloakPay does not retain them.

This means:
- CloakPay cannot freeze your funds (we do not hold them)
- CloakPay cannot reverse a transaction that has settled on Base
- If you lose access to your private key and have no backup, the funds are inaccessible permanently

This is the nature of self-custody. It is the strongest form of asset protection available, and it comes with the responsibility of key management. See [Key Management](../accounts/key-management.md).

## On-chain spending policy enforcement

Agent wallet spending limits are enforced by an on-chain Base smart contract, not by application logic. A spending policy violation cannot be bypassed by:

- Calling the API with a modified request
- Using the Base wallet key to submit a transaction directly
- Any other method

The on-chain smart contract evaluates the policy rules before executing the transfer instruction. If any rule is violated, the instruction is rejected and the transaction is never submitted to the validator network.

## Zero-knowledge privacy

Confidential transfers use zero-knowledge proofs on Base, which applies ElGamal encryption to USDC token balances and Bulletproof zero-knowledge proofs to validate transfers. These are peer-reviewed cryptographic primitives implemented in the Base token smart contract.

Key properties:
- Proof generation is client-side. Plaintext amounts never leave the user's device
- Base validators verify proofs mathematically without learning amounts
- Only sender and receiver can decrypt the amounts on their respective transactions
- Selective disclosure proofs are signed by the account holder's key and verifiable independently of CloakPay's servers

## Compliance framework

CloakPay operates under the following regulatory requirements:

### KYC and AML

All personal accounts require identity verification at signup. Verification is tiered based on transaction volume (see [Personal Accounts](../accounts/personal.md)). AML screening runs on every transaction using a third-party screening provider.

### GENIUS Act (2025)

USDC is a qualifying regulated stablecoin under the GENIUS Act (Guiding and Establishing National Innovation for US Stablecoins Act, passed July 2025). CloakPay's USDC operations are structured to comply with this framework, including reserve requirements and issuer accountability.

### Travel Rule

For transfers above the applicable threshold, CloakPay collects and transmits required counterparty information. Confidential transfers satisfy the travel rule via selective disclosure proofs, which allow transaction details to be shared with obligated parties without exposing the full account history.

### Jurisdictional availability

CloakPay is available in the United States and select EU member states during the beta period. Users in jurisdictions with explicit prohibitions on stablecoin or crypto payments are blocked at the account creation stage.

## Address screening

Every transfer recipient address is screened against sanctions lists (OFAC and equivalent EU/UK lists) before the transaction is submitted. Transfers to sanctioned addresses are blocked with an error code of `recipient_sanctioned`.

## Webhook security

Webhook payloads are signed with HMAC-SHA256 using the webhook secret you configure. Always verify the `Cloakpay-Signature` header before processing events. See [Webhooks](../sdk/rest.md#webhooks).

## API key security

- API keys are shown exactly once at creation
- Rotate keys immediately if you suspect they have been compromised
- Agent wallet keys are scoped: they can only operate on the specific wallet they were issued for
- Use environment variables or a secrets manager to store keys. Do not commit them to source control

## Responsible disclosure

If you discover a security vulnerability in CloakPay, please report it to security@usecloakpay.xyz. We aim to respond within 48 hours. We do not pursue legal action against researchers who report vulnerabilities responsibly.
