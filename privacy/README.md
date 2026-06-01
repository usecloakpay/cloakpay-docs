# Privacy Model

CloakPay's privacy design is built around one core idea: your transfer amounts should be yours to share, not broadcast by default.

## The problem with public blockchains

On a standard Base transfer, three pieces of information are public: the sender address, the receiver address, and the amount. Anyone with the sender's wallet address can see every transaction they have ever made, including the amounts. For business payments, payroll, and personal transfers, this is not acceptable.

Most privacy solutions respond by hiding everything, including addresses. This creates a different problem: it makes the activity unauditable, attracts regulatory scrutiny, and is increasingly difficult to use in compliance-aware environments.

## CloakPay's approach: confidentiality, not anonymity

CloakPay hides amounts while keeping addresses visible. This is the "confidentiality, not anonymity" model.

**On-chain for a standard transfer:**
```
Sender address [visible]  ->  Amount [visible]  ->  Receiver address [visible]
```

**On-chain for a CloakPay confidential transfer:**
```
Sender address [visible]  ->  Amount [encrypted]  ->  Receiver address [visible]
```

The encrypted amount is accompanied by a zero-knowledge proof that cryptographically proves the transfer is valid (the sender has sufficient funds, the amount is positive) without revealing what the amount is.

This model is defensible with regulators, auditors, and institutional counterparties. You can prove a transaction happened and that it was valid, without exposing its value to the world.

## What is hidden

- Transfer amounts
- Your USDC balance (visible only to you)
- Fee amounts on confidential transfers

## What is not hidden

- Your wallet address (on-chain, always visible)
- The existence of a transaction (on-chain, always visible)
- Counterparty addresses (on-chain, always visible)
- Memos (encrypted between sender and receiver, not on-chain)

## Technical basis

Confidential transfers in CloakPay are built on zero-knowledge proofs on Base, part of the ERC-20 extensions standard. This extension uses homomorphic encryption (ElGamal) and zero-knowledge proofs (sigma protocols / Bulletproofs) to encrypt token amounts.

Key properties:
- Proof generation happens client-side, in your browser or app
- The encrypted amount and proof are submitted together to Base
- Base validators verify the proof without learning the amount
- Only the sender and receiver, holding their respective decryption keys, can read the amount

## Selective disclosure

You can prove the content of any specific transaction to a third party without exposing your full history. Generate a selective disclosure proof for any transaction from the app or API. The proof cryptographically demonstrates the amount, sender, receiver, and timestamp of that specific transaction. Share it with whoever needs to see it.

This is how you provide evidence to a tax authority, auditor, or counterparty without handing over your entire account history.

[Selective Disclosure](selective-disclosure.md)

## Compliance

CloakPay operates under a compliance-first privacy model. Accounts require KYC at creation. Address screening runs against sanctions lists on every transaction. The selective disclosure mechanism satisfies travel rule requirements.

CloakPay is designed to work within regulatory frameworks, not around them. The privacy features are engineered to protect users from each other and from data brokers, not to circumvent financial oversight.

[Security and Compliance](../security/README.md)
