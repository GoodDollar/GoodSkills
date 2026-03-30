# Check identity guide

Use when the user asks whether an address is eligible for UBI or how identity links wallets.

## GoodDocs alignment

- [Connect another wallet address to identity](https://docs.gooddollar.org/user-guides/connect-another-wallet-address-to-identity): associated addresses resolve to a verified root in the Identity contract; `connectAccount` links wallets.
- One claim per day applies across all connected addresses for the same verified identity (see the hint on that page).

## Goal

Determine whitelist or authentication status with deterministic on-chain reads.

## Required inputs

- `nameServiceAddress` or explicit Identity address
- `account` to check
- `rpcUrl` and chain configuration

## Execution flow

1. Resolve `IDENTITY` from NameService when used on the deployment.
2. Read `getWhitelistedRoot(account)` or equivalent for the deployed Identity version.
3. Treat non-zero root as tied to a whitelisted identity tree when that is the protocol rule for the deployment.
4. Optionally read `isWhitelisted` or timestamps if the ABI exposes them.

## Return shape

- `isWhitelisted` or equivalent boolean summary
- `whitelistedRoot` or equivalent
- optional metadata fields when available

## Failure handling

- NameService cannot resolve `IDENTITY`: stop and fix deployment inputs using [Core contracts](https://docs.gooddollar.org/for-developers/core-contracts).
- Read failures: return the failing call and next step.
