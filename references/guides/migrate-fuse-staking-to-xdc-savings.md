# Fuse to XDC staking migration guide

Use when the user wants to migrate an existing Fuse governance stake into the XDC Ubeswap GoodDollar savings contract.

## Goal

Close a user stake on Fuse, bridge the resulting G$ to XDC, and stake on XDC for that user in a controlled backend flow.

## Required inputs

- user address on Fuse and corresponding destination address on XDC
- Fuse `GovernanceStakingV2` address (`production` entry in deployment metadata)
- Fuse G$ token address and bridge contract address
- XDC G$ token address and Ubeswap savings contract address
- backend signer or service wallet with required execution permissions
- chain RPC URLs for Fuse and XDC

## Address resolution quick table

| Purpose | Network | Source key/path | Value |
|---|---|---|---|
| Governance staking (source close) | Fuse (`production`, `networkId: 122`) | `deployment.json` -> `production.GovernanceStakingV2` | `0xB7C3e738224625289C573c54d402E9Be46205546` |
| Governance staking (previous) | Fuse (`production`, `networkId: 122`) | `deployment.json` -> `production.GovernanceStaking` | `0xFAF457Fb4A978Be059506F6CD41f9B30fCa753b0` |
| Fuse G$ token | Fuse (`production`, `networkId: 122`) | `deployment.json` -> `production.GoodDollar` | `0x495d133B938596C9984d462F007B676bDc57eCEC` |
| Fuse bridge | Fuse (`production`, `networkId: 122`) | `deployment.json` -> `production.MpbBridge` | `0xa3247276DbCC76Dd7705273f766eB3E8a5ecF4a5` |
| Destination savings | XDC (`networkId: 50`) | Ubeswap savings deployment | `0x3BeaaC603b445C7E8AdC46B3867404e1Bde9E047` |

Canonical sources:

- [GoodProtocol deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json)
- [Fuse explorer contract (GovernanceStakingV2 address)](https://explorer.fuse.io/address/0xB7C3e738224625289C573c54d402E9Be46205546?tab=contract)
- [Ubeswap gooddollar-contracts](https://github.com/Ubeswap/gooddollar-contracts)

## Execution flow

1. Confirm the user granted sufficient allowance on Fuse G$ to the backend execution wallet if your migration path requires a pull-model transfer.
2. Verify current stake state on Fuse before closing:
   - stake token balance
   - withdrawable stake amount
   - pending rewards if any
3. Execute Fuse unstake or close flow on governance staking (`withdrawStake` or equivalent full-close path).
4. Compute net G$ available for migration after unstake completion and any reward claim behavior.
5. Bridge G$ from Fuse to XDC using the configured bridge path and track the transfer id or tx hash pair.
6. Wait for destination finalization on XDC and verify credited G$ balance at the backend execution wallet.
7. Approve XDC savings contract to spend migrated G$ amount.
8. Stake for the user on XDC with `stakeFor(amount, recipient)` on Ubeswap `GooddollarSavings`.
9. Return a migration result with both chain tx hashes and final XDC staked amount.

## Deterministic snippet

```js
import { ethers } from "ethers";

const fuse = new ethers.JsonRpcProvider(process.env.FUSE_RPC_URL);
const xdc = new ethers.JsonRpcProvider(process.env.XDC_RPC_URL);
const signerFuse = new ethers.Wallet(process.env.BACKEND_PK, fuse);
const signerXdc = new ethers.Wallet(process.env.BACKEND_PK, xdc);

const user = process.env.USER_ADDRESS;
const migrateAmount = BigInt(process.env.MIGRATE_AMOUNT);

const xdcGd = new ethers.Contract(
  process.env.XDC_GD_TOKEN,
  [
    "function approve(address spender,uint256 amount) returns (bool)",
    "function balanceOf(address) view returns (uint256)",
  ],
  signerXdc,
);

const savings = new ethers.Contract(
  process.env.XDC_SAVINGS,
  ["function stakeFor(uint256 amount,address recipient)"],
  signerXdc,
);

const approveTx = await xdcGd.approve(process.env.XDC_SAVINGS, migrateAmount);
await approveTx.wait();

const stakeTx = await savings.stakeFor(migrateAmount, user);
const receipt = await stakeTx.wait();

console.log(
  JSON.stringify(
    {
      user,
      xdcStakeTx: receipt.hash,
      migratedAmount: migrateAmount.toString(),
    },
    null,
    2,
  ),
);
```

## Pre-check failures

- User allowance missing on Fuse: stop and request allowance tx from user.
- Stake close fails on Fuse: stop and return exact revert reason before bridge.
- Bridge transfer not finalized on XDC: do not call `stakeFor` until destination balance is confirmed.
- XDC savings approval missing or too low: re-approve exact amount before staking.

## Output contract

- user address
- Fuse unstake tx hash
- bridge tx hash or transfer identifier
- XDC stake tx hash
- final staked amount on XDC
