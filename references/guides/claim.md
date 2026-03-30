# Claim guide

Use when the user wants to claim daily UBI. Protocol context: [How GoodDollar works](https://docs.gooddollar.org/how-gooddollar-works) and [UBIScheme](https://docs.gooddollar.org/for-developers/core-contracts/ubischeme).

## Goal

Execute a safe `claim()` with identity pre-checks and clear outputs.

## GoodDocs alignment

- UBI is distributed daily to verified users; the active pool is split among claimers in each period (see [How GoodDollar works](https://docs.gooddollar.org/how-gooddollar-works)).
- UBIScheme deployments vary by chain (Fuse, Celo, XDC on the core contracts page); resolve live addresses from [Core contracts](https://docs.gooddollar.org/for-developers/core-contracts) or [deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json).

## Required inputs

- `nameServiceAddress` or explicit UBIScheme and Identity addresses from deployment metadata
- `rpcUrl` and chain configuration
- signer context

## Execution flow

1. Resolve `IDENTITY` and `UBISCHEME` from NameService when NameService is the source of truth for the deployment.
2. Confirm whitelist status for the claiming account.
3. Optionally read entitlement or claimable state before sending `claim()`.
4. Call `claim()` on the resolved UBIScheme (contract generation may differ by deployment; align ABI with your target).
5. Return tx hash and claimed amount when derivable from events or balance delta.

## Pre-check failures

- Not whitelisted: stop and point the user to identity verification flows in GoodDocs.
- Missing contract address: stop; use [deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json) or the core contracts table.
- Zero entitlement: communicate that nothing is claimable in the current period without guessing amounts.

## Output contract

- network
- resolved contract addresses
- tx hash
- claim outcome details when available
