# Account Types

CloakPay has two account types that share the same financial infrastructure but serve different use cases.

## Personal accounts

A personal account is a human-owned, non-custodial USDC account. You create it with an email and passkey, and your private key is derived from your device credential. CloakPay never sees your key.

Personal accounts are the foundation of the CloakPay product: a clean, private wallet for sending and receiving USDC, with a readable transaction feed, confidential transfers on by default, and a CloakPay handle for easy addressing.

[Personal Accounts](personal.md)

## Agent accounts

An agent account is a programmatically provisioned wallet designed to be used by autonomous software. Agent accounts are created via the API, owned by a parent personal account, and governed by a spending policy that determines what the agent can spend and when.

Agent accounts have their own Base addresses and USDC balances. They are funded by their operator (the parent account holder) and can transact independently within the bounds of their policy.

Agent accounts are identified as `@username/agent-name`, making the ownership relationship explicit on every transaction.

[Agent Accounts](agent-accounts.md)

## Comparison

| | Personal | Agent |
|---|---|---|
| Created via | Web app / mobile | API |
| Key derivation | Passkey (device) | API-provisioned |
| Balance | USDC + ETH reserve for gas | USDC + ETH reserve for gas |
| Spending policy | Not applicable | Required |
| Transactions | Manual (user-initiated) | Autonomous (within policy) |
| Handle format | `@username` | `@username/agent-name` |
| x402 support | No | Yes |
| HITL controls | Not applicable | Configurable |
