# CloakPay Documentation

CloakPay is a privacy-first crypto neobank built on Base. It gives humans and AI agents a shared financial account where transfers are confidential by default, every transaction settles on-chain in under a second, and you always hold your own keys.

## What makes CloakPay different

Most financial products make a binary choice: either everything is public (most crypto), or everything is hidden behind a custodian (traditional banks). CloakPay takes a third path.

**Confidential by default.** Transfer amounts are encrypted using zero-knowledge proofs on Base. Sender and receiver addresses remain on-chain and visible. Amounts are not. You can selectively disclose any transaction to an auditor, tax authority, or counterparty without exposing your full history.

**Agent-native from day one.** CloakPay accounts are programmable. You can provision a wallet for an AI agent via a single API call, attach a spending policy that enforces limits at the on-chain smart contract level, and receive real-time webhook events for every transaction. Agents transact autonomously within the rules you set.

**Non-custodial.** CloakPay never holds your private keys. Keys are derived from your device passkey and never leave your device unencrypted. You can export them at any time.

**Settled on Base.** Transactions finalize in under 2 seconds at a fraction of a cent. Every transfer is independently verifiable on Basescan.

## Who this is for

CloakPay is built for two audiences that increasingly overlap:

**Individuals and businesses** who want a clean, private stablecoin account for payments, payroll, or cross-border transfers. If you've used Revolut or Cash App but want your transaction history to stay yours, CloakPay is built for you.

**Developers and AI builders** who need programmable financial infrastructure for autonomous systems. If your agents are paying for API calls, settling invoices, or executing on-chain logic, CloakPay gives them a wallet that works the same way yours does.

## Beta

CloakPay is currently in public beta. Core features are stable and production-ready. Some advanced features are marked accordingly throughout these docs.

{% hint style="info" %}
The first 500 accounts created during beta pay zero fees. [Apply for early access](https://usecloakpay.xyz).
{% endhint %}

## Quick links

- [Quickstart](getting-started/quickstart.md) - Set up your first account and send a transfer in five minutes
- [Core Concepts](getting-started/concepts.md) - Understand accounts, privacy, and policies before you build
- [REST API](sdk/rest.md) - Complete HTTP API reference for all CloakPay endpoints
- [Agent Accounts](accounts/agent-accounts.md) - Provision wallets for AI agents via API

## Links

- Website: https://usecloakpay.xyz
- X: https://x.com/usecloakxyz_
- GitHub: https://github.com/usecloakpay
- Docs: https://docs.usecloakpay.xyz
