# Development

This repository is a scaffold. It has no working contracts, services, CLI, or demo UI yet.

## Layout

- `contracts/`: Rust workspace for registries, wallet, demo service, and shared types.
- `packages/`: TypeScript SDK and CLI placeholders.
- `services/`: Verification, submission, and history placeholders.
- `apps/demo/`: Demo application placeholder; no UI framework selected yet.
- `tests/`: Integration tests, adversarial tests, and fixtures.
- `deployments/testnet/`: Future deployment records.

## Local checks

Install Node.js with npm and a Rust toolchain with rustfmt. From the repository root:

```sh
npm install
npm run typecheck
npm run build
cargo check --workspace
cargo fmt --all -- --check
```

TypeScript is pinned to the version used to check this scaffold. Rust crates currently have no external dependencies. Select and pin the Soroban SDK and Stellar JavaScript SDK when contract integration begins.

These checks validate the scaffold only. There are no application tests or deployment commands yet. No network credentials are needed.
