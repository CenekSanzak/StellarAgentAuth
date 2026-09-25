## Stellar Agent Auth

The idea for this project came while building **Algoria** during the **Stellar Pro Hackathon**, where we worked with a local wallet and started thinking more deeply about how AI agents should safely interact with Stellar accounts.

While exploring this problem, we came across **ERC-8126** and **ERC-8196**, which introduce ideas around AI agent verification and policy-based authorization on Ethereum.

- ERC-8126: https://eips.ethereum.org/EIPS/eip-8126
- ERC-8196: https://eips.ethereum.org/EIPS/eip-8196

Today, an AI agent usually needs direct wallet access to perform transactions. This creates a security problem because the agent may receive more control than it actually needs.

The idea is to build a **Stellar-native smart wallet for AI agents** using Soroban.

Instead of giving an AI agent full control of a wallet, the user can give it limited and revocable permissions, such as:

- which assets it can use,
- how much it can spend,
- which addresses or contracts it can interact with,
- how long the permission is valid,
- and when the permission should be revoked.

For example, a user could allow an AI agent to spend up to 50 USDC per transaction, with a daily limit of 200 USDC, only for approved actions.

The goal is not to directly port the Ethereum standards, but to explore how these ideas can be implemented naturally on Stellar using **Soroban contract accounts, programmable authorization, and Stellar assets**.
