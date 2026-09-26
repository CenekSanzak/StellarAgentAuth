# Stellar AI Agent Verification & Authenticated Wallet

The idea for this project came while building **Algoria during the Stellar Pro Hackathon**, where we worked with a local wallet and started exploring how AI agents could safely interact with Stellar accounts.

During this work, we came across two recently finalized Ethereum standards:

- **ERC-8126 — AI Agent Verification**  
  https://eips.ethereum.org/EIPS/eip-8126

- **ERC-8196 — AI Agent Authenticated Wallet**  
  https://eips.ethereum.org/EIPS/eip-8196

Together, these ERCs introduce a model where AI agents can be verified and then given limited, policy-based authority to perform transactions without receiving unrestricted control over a user's wallet.

The project proposes to **implement these concepts natively on Stellar**, adapting the Ethereum-specific parts of the standards to Stellar accounts, Soroban smart contracts, Stellar assets, and programmable authorization.

The goal is to explore a Stellar-compatible implementation where AI agents can be verified and granted limited permissions such as spending limits, approved assets, approved contracts or addresses, expiration times, and revocation.

Rather than simply copying the Solidity implementations, the project would preserve the core behavior of ERC-8126 and ERC-8196 while designing the equivalent interfaces and flows around Stellar's architecture.

## High-Level Architecture

```text
User / Wallet Owner
        ↓
Agent Verification Layer
(ERC-8126 inspired)
        ↓
Policy & Permission Layer
(ERC-8196 inspired)
        ↓
Soroban Smart Wallet
        ↓
Stellar Assets / Contracts
        ↓
Events & Audit History
```

The verification layer checks the AI agent's identity or attestations. The policy layer defines what the agent is allowed to do, such as spending limits, allowed assets, approved addresses, expiration, and revocation. The Soroban smart wallet enforces these rules before executing transactions.

## Grant Deliverables

- **Stellar-compatible ERC-8126 implementation** for AI agent verification and attestations.
- **Stellar-compatible ERC-8196 implementation** for policy-based AI agent wallet authorization.
- **Soroban smart wallet contracts** with spending limits, allowlists, expiration, and revocation.
- **Agent verification + execution flow** demonstrating Verify → Authorize → Execute.
- **TypeScript SDK / CLI** for integrating and testing the contracts.
- **Testnet deployment and demo application** showing allowed and rejected agent transactions.
- **Open-source repository, tests, and documentation** for other Stellar developers.
