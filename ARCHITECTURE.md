# Architecture

This is the proposed design for Stellar AI Agent Verification & Authenticated Wallet. It lets an owner give an AI agent limited access to funds without sharing the owner's private key.

## What we take from the EIPs

[ERC-8126](https://eips.ethereum.org/EIPS/eip-8126) describes off-chain agent verification. Providers resolve an agent's registered metadata, assess applicable token, media, code, web, and wallet risks, and produce a risk score from 0 to 100. Lower scores mean lower risk. It also describes privacy proofs and uses ERC-8004 identities.

[ERC-8196](https://eips.ethereum.org/EIPS/eip-8196) describes policy-controlled execution. The wallet checks current verification before each action, enforces the owner's limits, and keeps an audit history linked by hashes. Its `minVerificationScore` is actually an upper risk limit; we call this `max_risk_score`.

We adapt these ideas to Stellar addresses, Soroban authorization, and Stellar assets. The first release implements a defined subset, not full ERC compatibility. Privacy proofs, entropy commit-reveal, and TLS credential delegation are future work.

## Main components

```text
Owner → Register agent → Verify agent → Approve policy
                                             ↓
Agent → Sign action → Soroban wallet → Approved asset or contract
                          ↓
                   Events and history
```

| Component | Role |
| --- | --- |
| Agent registry | Stores the agent ID, controller, signing address, and metadata reference. |
| Verification service | Checks the agent off-chain and publishes its result. |
| Verification registry | Stores provider-authenticated results, risk scores, expiry, and revocation status. |
| Smart wallet | Holds funds, checks permissions, and executes allowed actions. |
| SDK, CLI, and demo | Let owners manage policies and agents submit actions. |
| History service | Reads contract events and displays activity. |

Contracts use Rust and Soroban. The SDK, CLI, and services use TypeScript. Policy checks live inside the wallet so checking limits and moving funds happen in one transaction.

## Identity and verification

An agent's controller registers its signing address and a metadata URI with a content hash. Changes to the key or metadata create a new identity revision and require fresh verification and permission.

The verification service reads that registered metadata and performs the supported checks. It publishes the agent ID, revision, provider, risk score, completed checks, evidence hash, and expiry. Private reports stay off-chain; an evidence hash is not a privacy proof.

Each policy names a provider the owner trusts. Missing, expired, revoked, or failed required checks block execution. A newer failure cannot be bypassed by presenting an older passing result. Serious wallet-risk flags also block execution even when the average score is low.

The first release uses a Stellar agent registry. ERC-8004 integration and broader verification coverage can follow.

## Wallet and permissions

The owner deposits assets into a separate wallet contract. The agent can spend only those funds under an approved policy.

A policy defines:

- The agent identity, revision, and signing address.
- The trusted provider, required checks, and maximum risk score.
- Allowed assets, actions, contracts, and recipients, plus explicit blocked targets.
- Per-transaction and daily spending limits for each asset.
- Start time, expiry, and revocation status.

Amounts use integer asset units. Different assets have separate budgets. Daily limits reset at midnight UTC using ledger time; separate policies have separate budgets.

Only the owner can create or revoke policies, pause agent access, or withdraw funds. Policies cannot be edited: the owner revokes and replaces them. Empty allowlists permit nothing, and explicit blocks take priority.

## Execution flow

1. The agent builds an action containing the wallet, policy hash, destination, asset, amount, deadline, and a unique sequence number.
2. The SDK prepares Soroban authorization for the exact request. The agent signs it; a submitter may pay the transaction fee.
3. The wallet authenticates the agent and checks the policy, sequence number, deadline, and current verification result.
4. It checks the action and spending limits, updates its counters, and calls the approved asset or contract.
5. On success, it records an event. If the call fails, the transaction rolls back, including the counters.

Contract integrations use explicit methods and validated arguments. Allowing a contract address does not grant access to every method or token approval.

Revocation takes effect once confirmed on-chain. It blocks later executions, including previously signed requests, but cannot undo completed transfers.

## History and storage

Successful executions and owner changes emit events. Each audit entry includes the previous entry's hash, with the latest hash stored in the wallet. The history service uses these events to show activity and detect gaps or changes.

Rejected requests appear through simulation errors or transaction results; failed transactions do not leave committed wallet events.

Policies, revocations, counters, and sequence numbers use persistent storage. Maintenance extends Soroban storage lifetimes and handles restoration. Restoring data never resets limits or makes an expired policy valid again.

## First release

The testnet demo covers agent registration, provider verification, wallet funding, policy approval, asset transfers, and one restricted contract integration.

Tests must show that valid actions succeed and that overspending, unapproved destinations, stale verification, replayed requests, and revoked policies fail. Signed integration tests also check that failed calls leave balances and spending counters unchanged.

Verification is a trust signal, not a guarantee of safe behavior. The wallet's limits remain the final control over every agent action.
