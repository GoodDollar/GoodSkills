# Save and stake guide

Use when the user wants to stake G$, withdraw rewards, or exit stake. Staking economics sit alongside other protocol allocations described in [How GoodDollar works](https://docs.gooddollar.org/how-gooddollar-works).

## GoodDocs alignment

- Token integration and fee awareness: [How to integrate the G$ token](https://docs.gooddollar.org/for-developers/developer-guides/how-to-integrate-the-gusd-token) (`_processFees`, decimals per chain).
- Contract addresses: [Core contracts](https://docs.gooddollar.org/for-developers/core-contracts) and [deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json).

## Goal

Run staking actions with balance and allowance safety checks.

## Required inputs

- `nameServiceAddress` or explicit staking and token addresses
- `amount` or `shares` depending on the action
- `rpcUrl`, chain configuration, signer

## Execution flow

1. Resolve staking and G$ token addresses from NameService or deployment metadata.
2. Read token balance and allowance.
3. Approve the staking contract when `stake` uses `transferFrom`.
4. Execute `stake`, `withdrawRewards`, or `withdrawStake` as requested.
5. Return tx hash and key resulting balances or events.

## Failure handling

- Insufficient balance: report shortfall.
- Approval issues: report token, spender, and required allowance.
- Reverts: return attempted function and parameters without guessing custom errors.
