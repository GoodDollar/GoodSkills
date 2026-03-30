# Inviter and invitee reward model

On-chain behavior is implemented in GoodProtocol **`InvitesV2`** ([`contracts/invite/InvitesV2.sol`](https://github.com/GoodDollar/GoodProtocol/blob/master/contracts/invite/InvitesV2.sol)). GoodDocs may not have one dedicated page; always confirm addresses from [deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json).

## Core mechanics

- Users register with **`join(bytes32 _myCode, bytes32 _inviterCode)`**: binds an invite code to `msg.sender`, optionally records an inviter from `codeToUser[_inviterCode]`, emits **`InviteeJoined`**. Reverts include **`self invite`**, **`invite code already in use`**, **`user already joined`** when state forbids updates.
- **`canCollectBountyFor(address invitee)`** (view): eligibility uses configurable **`minimumDays`** and **`minimumClaims`** (set via **`setMinimums`**), reads **`UBISchemeV2.totalClaimsPerUser`** when available, requires invitee **whitelisted**, **`bountyAtJoin > 0`**, not yet **`bountyPaid`**, inviter whitelisted (or zero / campaign **`address(this)`** paths), and **same chain** as identity’s `getWhitelistedOnChainId` when that call succeeds.
- **`bountyFor(address invitee)`**: pays bounty when **`canCollectBountyFor`** holds; reverts **`user not elligble for bounty yet`** otherwise. Internally **`_bountyFor`** transfers G$ to inviter and invitee per level rules and emits **`InviterBounty`**.
- **`collectBounties()`**: inviter iterates **`pending`** invitees and batches eligible payouts; gas limits apply inside the loop.

## Events

- **`InviteeJoined(address indexed inviter, address indexed invitee)`**
- **`InviterBounty(address indexed inviter, address indexed invitee, uint256 bountyPaid, uint256 inviterLevel, bool earnedLevel)`**

## Useful views

- **`users(address)`**, **`codeToUser(bytes32)`**, **`getPendingInvitees`**, **`getPendingBounties`**, **`levels`**, **`minimumClaims`**, **`minimumDays`**, **`active`**.

## GoodDocs anchors

- Linked wallets and identity: [Connect another wallet address to identity](https://docs.gooddollar.org/user-guides/connect-another-wallet-address-to-identity).
- Broader ecosystem incentives: [Developer guides](https://docs.gooddollar.org/for-developers/developer-guides).

## What agents should do

1. Resolve **`InvitesV2`** (or deployed name) for the chain from `deployment.json`.
2. Read **`canCollectBountyFor`** and **`users`** for both parties before asserting payouts.
3. Surface exact revert strings and eligibility flags when a bounty is not available.

## Failure cases

- Self-invite or duplicate code (`join` reverts).
- Invitee not whitelisted or **`minimumClaims` / `minimumDays`** not met.
- **`bountyPaid`** already true or **`bountyAtJoin`** zero.
- Contract **`active == false`** or wrong chain relative to identity.
