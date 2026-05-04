# Why Fuse to XDC staking migration uses a staged backend flow

This explains why the migration is split into allowance detection, unstake, bridge, and destination re-stake instead of a single transaction. In this context, Fuse `GovernanceStakingV2` is the old/source staking contract and XDC `GooddollarSavings` is the new/destination staking contract.

## Why this cannot be one-chain atomic

Fuse governance staking and XDC savings live on different chains, so the system cannot atomically close and reopen stake in one EVM transaction. Cross-chain migration is asynchronous by design and must tolerate timing gaps between source completion and destination finalization.

## Why user allowance is first

The migration flow assumes a backend-operated execution wallet. If that wallet needs to pull or operate on user funds in the source path, user approval must exist first. Without allowance, all downstream steps fail, so approval state is the earliest hard gate.

## Why unstake is separated from bridge

The source staking position is the canonical balance record on Fuse. The migration must first materialize transferable G$ by closing or reducing stake, then bridge only the confirmed unlocked amount. Bridging before final unstake confirmation introduces mismatch risk.

## Why bridge finalization must be explicit

Bridge transfers are eventually consistent across chains. The destination stake step must only run after the bridged G$ is confirmed on XDC. This avoids phantom staking attempts and preserves deterministic accounting.

## Why destination uses `stakeFor`

Ubeswap `GooddollarSavings` supports `stakeFor(amount, recipient)`, which allows the backend execution wallet to stake on behalf of the user after bridge finalization. This fits migration operations where custody is temporary during the transfer window.

## Main operational risks

- partial migration from source unstake or bridge limit constraints
- stuck-in-transit bridge messages delaying destination staking
- stale assumptions about contract addresses across networks
- reward expectation mismatch when moving from Fuse governance staking mechanics to XDC savings mechanics

## Contract/source map

- Fuse staking family (GoodProtocol): [`GovernanceStaking.sol`](https://github.com/GoodDollar/GoodProtocol/blob/master/contracts/governance/GovernanceStaking.sol)
- Fuse deployment mapping: [`deployment.json`](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json) (`production.GovernanceStakingV2`)
- XDC savings implementation (Ubeswap): [`GooddollarSavings.sol`](https://github.com/Ubeswap/gooddollar-contracts/blob/main/contracts/GooddollarSavings.sol)
- Ubeswap contracts repository: [Ubeswap/gooddollar-contracts](https://github.com/Ubeswap/gooddollar-contracts)
