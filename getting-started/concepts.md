# Core Concepts

This page covers the foundational ideas behind CloakPay. Understanding these concepts will help you make good decisions when building with the API.

## Accounts

CloakPay has two account types: personal accounts and agent accounts. They share the same underlying infrastructure but serve different purposes.

**Personal accounts** are for humans. You sign in with a passkey, hold USDC and ETH, and interact via the web app or mobile app. Your private key is derived from your passkey and never leaves your device.

**Agent accounts** are for software. They are provisioned programmatically via the API, owned by a parent human account, and designed to transact autonomously. An agent account has its own Base wallet address, its own USDC balance, and a spending policy that governs what it can and cannot do.

Agent account IDs are namespaced to their parent: `@yourhandle/agent-name`. This makes it clear which human account is responsible for an agent's activity.

See [Account Types](../accounts/README.md) for the full breakdown.

## Wallets and Keys

CloakPay is non-custodial. You hold your own private keys. The platform never has access to them.

For personal accounts, your private key is derived from your device passkey using standard key derivation (PBKDF2/WebAuthn). If you want to use CloakPay on multiple devices or with an external wallet, you can export your key at any time.

For agent accounts, keys are generated server-side at provisioning and returned to you once. CloakPay does not store them after that. You are responsible for key storage for agent wallets.

See [Key Management](../accounts/key-management.md) for rotation, recovery, and export.

## USDC and Gas

CloakPay accounts are denominated in USDC (Circle, native Base issuance). ETH is used only for transaction fees and is maintained automatically.

When you send USDC, CloakPay handles the ETH gas fee transparently. A small ETH reserve is held in each wallet for this purpose. When the reserve drops below the threshold for one transaction, it is automatically topped up via an internal swap. You never interact with ETH directly unless you choose to.

## Confidential Transfers

On a standard Base transfer, the amount is visible on-chain to anyone who looks. CloakPay defaults to confidential transfers, which hide the amount while keeping sender and receiver addresses visible.

This is built on zero-knowledge proofs on Base, which uses homomorphic encryption and zero-knowledge proofs. The ZK proof is generated client-side (in your browser or app) so the amount never leaves your device unencrypted.

The key distinction: confidentiality, not anonymity. Your address is visible on-chain. The amount is not. This is the model that regulators and institutions can work with, and it is the model CloakPay uses.

You can disable confidential mode per-transfer if you want the amount to be public. You can also generate a selective disclosure proof to share specific transactions with auditors or counterparties without revealing your full history.

See [Privacy Model](../privacy/README.md) for more.

## Spending Policies

Every agent account is bound to a spending policy at creation. Policies define the rules under which an agent can transact: maximum per transaction, maximum per day, velocity caps, allowed recipient addresses, and time windows.

Policies are enforced at the on-chain smart contract level before any transfer instruction executes. This is not an application-layer check that can be bypassed by calling the API differently. The on-chain smart contract rejects any transaction that violates the policy, regardless of how the request was formed.

Policy changes take effect within one Base block, approximately 2 seconds.

See [Spending Policies](../policies/README.md) for the full parameter reference.

## x402 Payments

x402 is an open payment standard for machine-to-machine transactions. It lets an AI agent pay for a resource (an API call, a compute job, a data feed) directly, without a credit card, OAuth token, or human in the loop.

CloakPay implements x402 natively. Base accounts for approximately 65% of all x402 transactions in the wild. When an agent makes a request to an x402-enabled API, the API returns a `402 Payment Required` response with a payment descriptor. The agent's CloakPay wallet handles the payment and retries the request automatically.

See [x402 Protocol](../agents/x402.md) for implementation details.

## Human-in-the-Loop Controls

Agents transact autonomously within their spending policy. For transactions above a threshold you define, CloakPay can surface an approval request to the parent account holder before proceeding.

Below the threshold: the agent executes immediately without interruption.
Above the threshold: the agent pauses and sends a push notification to the parent account. The human approves or rejects via app or API. The agent continues or aborts accordingly.

This gives you fine-grained control over when you want to supervise agent spend and when you want to stay out of the way.

See [Human-in-the-Loop Controls](../agents/human-in-the-loop.md) for configuration.

## On-Chain Settlement

Every CloakPay transaction is a real Base transaction. There is no internal ledger that shadows the chain. Every transfer you see in your feed has a corresponding Base transaction hash that you can verify independently on Basescan.

Finality is typically in under 2 seconds. There is no "pending" state that lasts minutes or hours. Either the transaction confirms or it fails, and CloakPay surfaces the result to you in real time.

## CloakPay Handles

Every account gets a human-readable handle in the format `@username`. You can send to a handle instead of a raw Base wallet address. Handles resolve to the account's current Base address.

Agent accounts are namespaced: `@username/agent-name`. You can also send to raw Base addresses for interoperability with any external wallet.
