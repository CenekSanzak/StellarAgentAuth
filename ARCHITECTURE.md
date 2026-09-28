# Architecture

This project lets a wallet owner give an AI agent limited permission to use funds on Stellar. The owner keeps control of the wallet, while the agent can perform only the actions the owner has approved.

**Current status:** The repository contains folders, package configuration, and placeholder modules. The behavior below is the planned design; contracts and services are not implemented or deployed yet.

## Design overview

The system separates two decisions: whether an agent has passed verification, and whether a particular action is allowed. Passing verification does not give an agent permission to spend.

```text
Agent controller → Agent registry → Verification service
                                            ↓
                                    Verification registry
                                            ↑
Wallet owner → Approve policy → Smart wallet ← Signed agent action
                                    ↓
                           Approved asset or contract
                                    ↓
                            Events → History service
```

The agent controller manages the agent's identity. The wallet owner controls the funds and permissions. These may be different people.

The design draws from [ERC-8126](https://eips.ethereum.org/EIPS/eip-8126) for agent verification and [ERC-8196](https://eips.ethereum.org/EIPS/eip-8196) for policy-based execution. It adapts their concepts to Stellar rather than copying Ethereum interfaces. The first release covers a subset of their features and does not claim full ERC compatibility.

## Components and responsibilities

Contracts are planned in Rust with Soroban. The SDK, services, CLI, and demo use TypeScript.

| Location | Responsibility |
| --- | --- |
| `contracts/agent-registry` | Store agent identities, signing addresses, and metadata references. |
| `contracts/verification-registry` | Store authenticated provider results and their validity. |
| `contracts/smart-wallet` | Hold funds, enforce policies, execute actions, and record events. |
| `contracts/shared-types` | Define common identity, policy, verification, and error types. |
| `contracts/demo-service` | Provide one restricted contract integration for the demo. |
| `services/verifier` | Read agent metadata, run checks, and publish results. |
| `services/submitter` | Simulate transactions, submit them, and track their status. |
| `services/indexer` | Read events and build a searchable audit history. |
| `packages/sdk` | Provide reusable clients for these workflows. |
| `packages/cli` and `apps/demo` | Let developers and owners interact with the system. |

Policy checks remain inside the wallet contract. This keeps permission checks, spending counters, and fund movements in the same transaction.

## Agent identity and verification

The controller registers an agent ID, signing address, and metadata URI with a content hash. The verifier fetches the registered metadata and checks that its content matches the hash. Changing the signing key or metadata creates a new identity revision, requiring fresh verification and owner permission.

ERC-8126 describes off-chain checks covering token, media, code, web, and wallet risks. It uses a risk score from 0 to 100, where lower means lower risk, and builds on ERC-8004 identities. Our first release uses a Stellar registry and clearly identifies which checks are supported.

The verifier publishes a result containing the agent ID and revision, provider, completed checks, risk score, evidence hash, issue time, expiry, and revocation status. Detailed reports stay off-chain. An evidence hash links a result to a report; it is not a zero-knowledge proof.

Each policy names a provider the owner trusts. Before every action, the wallet reads that provider's latest result. Missing, expired, revoked, or failed required checks block execution. A newer failure cannot be replaced with an older passing result. Serious wallet-risk flags also block execution regardless of the average score.

## Wallet and permissions

The owner deposits supported assets into a separate wallet contract. The agent receives no signing authority over the owner's ordinary Stellar account and never needs the owner's private key.

A policy defines:

- **Who:** agent ID, identity revision, and signing address.
- **Verification:** trusted provider, required checks, maximum result age, and maximum risk score.
- **Actions:** allowed assets, methods, contracts, and recipients, with explicit blocked targets.
- **Limits:** per-transaction and daily spending caps for each asset.
- **Lifetime:** activation time, expiry, and revocation status.

The risk threshold is called `max_risk_score`. This preserves ERC-8196's rule that scores above the policy threshold are rejected.

Amounts use integer asset units, and assets are identified by contract address. Daily budgets reset at midnight UTC using ledger time. Each policy has its own budget, so the owner interface must show the combined exposure when approving several policies.

Only the owner can approve or revoke policies, pause agent access, or withdraw funds. Permission changes require replacing a policy. Empty allowlists permit nothing, and explicit blocks take priority.

## How an action executes

1. The agent prepares an action with the wallet address, policy hash, action details, deadline, and next policy sequence number. For a transfer, the details include the asset, recipient, and amount.
2. The SDK prepares authorization for that exact request. The agent signs it, and a submitter may pay the transaction fee. Paying the fee gives the submitter no spending authority.
3. The wallet authenticates the agent and checks that the policy is active, the identity revision matches, and the request has not expired or already been used.
4. It reads the current verification result, validates the action, and checks the remaining budget.
5. It advances the sequence number, updates spending counters, and calls the approved asset or contract. Success records an audit event; failure rolls back the action and counters together.

Contract integrations allow specific methods and validated arguments. An approved contract address does not grant unrestricted access to its methods or token approvals. The first release handles one action per request.

Revocation blocks subsequent execution once confirmed on-chain, including requests signed earlier. It cannot undo a completed transfer.

## History, storage, and trust

Successful actions and owner changes emit events. Audit entries include the previous entry's hash, and the wallet stores the latest hash and sequence. The indexer reconstructs this history and can detect missing or altered entries. It has no authority to approve transactions.

Rejected actions are shown through simulation errors or transaction results. Failed transactions do not leave committed wallet events.

Policies, revocations, spending counters, and sequence numbers use persistent storage. Storage maintenance must extend Soroban data lifetimes and handle restoration. Restoring data must preserve spending history and revocation; it must never reactivate an expired policy.

Verification depends on the selected provider and does not guarantee safe behavior. If an agent key is compromised, an attacker may use its remaining permissions. Narrow policies and owner revocation limit that exposure.

## First release and validation

The testnet demo will show registration, verification, wallet funding, policy approval, asset transfers, one restricted contract integration, and revocation.

`tests/integration` will cover signed workflows and transaction rollback. `tests/adversarial` will cover replay, invalid authorization, overspending, unapproved destinations, stale verification, and revoked policies. Public fixtures belong in `tests/fixtures`; deployment records belong in `deployments/testnet`.

Privacy proofs, entropy commit-reveal, TLS credential delegation, external identity registry integration, and arbitrary contract execution are outside the first release. Each requires a separate design before implementation.
