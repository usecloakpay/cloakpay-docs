# Agent Payments

CloakPay treats AI agents as first-class account holders. An agent has its own wallet address, its own USDC balance, and its own transaction history. It can pay for services, transfer funds to other agents, and settle invoices without human involvement.

## The model

Every agent account is:

- **Provisioned by a human.** The parent account holder creates the wallet via API and is accountable for its activity.
- **Funded by its operator.** The agent's balance is USDC. The operator funds it by sending USDC to the agent's wallet address or handle.
- **Governed by a policy.** The spending policy is enforced on-chain. The agent cannot exceed its limits regardless of what the code tells it to do.
- **Auditable.** Every agent transaction appears in the parent account's feed, tagged with the agent's name.

## What agents can do

**Pay for external services.** Using the x402 protocol, an agent can pay for API calls, compute resources, and data feeds directly in USDC, settling in under a second.

**Transfer to other agents.** An orchestrator agent can fund sub-agents for specific tasks. The sub-agents complete their tasks, and unused balances can be returned.

**Operate continuously.** There is no session to maintain. An agent with a valid API key and a funded wallet can transact 24/7 without re-authentication.

## What agents cannot do

- Exceed their spending policy limits (enforced on-chain)
- Transfer to blocked recipients
- Continue operating after the policy expiry date
- Execute transactions above the co-sign threshold without human approval

## Lifecycle

```
Operator creates policy
        |
Operator provisions agent wallet (with policy)
        |
Operator funds wallet
        |
Agent receives credentials (scoped API key)
        |
Agent transacts autonomously within policy
        |
Operator monitors feed + webhooks
        |
Operator adjusts policy / funds / pauses / closes
```

## Getting started

- [Agent Accounts](../accounts/agent-accounts.md) - Create and manage agent wallets
- [Spending Policies](../policies/README.md) - Define what agents can spend
- [x402 Protocol](x402.md) - Enable agents to pay for services automatically
- [Human-in-the-Loop Controls](human-in-the-loop.md) - Configure when humans need to approve
