# Inviter and invitee reward model

On-chain behavior is implemented in GoodProtocol **`InvitesV2`** ([`contracts/invite/InvitesV2.sol`](https://github.com/GoodDollar/GoodProtocol/blob/master/contracts/invite/InvitesV2.sol)). Bounty eligibility calls into **`Identity`** (for example **`IdentityV4`**) for **`isWhitelisted`** and **`getWhitelistedOnChainId`**; whitelist and reverification rules live on the identity contract, not inside **`InvitesV2`**. GoodDocs may not have one dedicated page; always confirm addresses from [deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json).

## Identity whitelist (how `isWhitelisted` affects invites)

The following matches GoodProtocol **`IdentityV4`** ([`contracts/identity/IdentityV4.sol`](https://github.com/GoodDollar/GoodProtocol/blob/master/contracts/identity/IdentityV4.sol)).

### Stored per-address state

Each address has an **`Identity`** record: **`dateAuthenticated`**, **`dateAdded`**, **`did`**, **`whitelistedOnChainId`**, **`status`**, **`authCount`**. **`status`** means: **`0`** unused, **`1`** normal whitelist, **`2`** DAO-registered contract, **`255`** blacklisted.

### What “whitelisted” means for `isWhitelisted(account)`

1. If **`identities[account].status == 1`**, the contract does **not** return **`true`** whenever the account is in a **reverification-due** window. It computes **`daysSinceAuthentication`** from **`block.timestamp`** and **`dateAuthenticated`**, then returns **`true`** only if **`shouldReverify(account, daysSinceAuthentication)`** is **`false`**. So a user can be “on the list” with status 1 but still **`isWhitelisted == false`** until an admin calls **`authenticate` / `authenticateWithTimestamp`** again (see reverification below).

2. If there is no local status-**`1`** record, **`isWhitelisted`** falls back to **`oldIdentity.isWhitelisted(account)`** when **`oldIdentity`** is set, with try/catch returning **`false`** on failure.

3. **`InvitesV2.canCollectBountyFor`** uses **`getIdentity().isWhitelisted(_invitee)`** (and the same for the inviter when present). That is evaluated on the **specific address** passed in (the invitee’s joined address). It does **not** substitute **`getWhitelistedRoot`**; if product flows use a “connected” wallet for claiming, the invitee address used in **`join`** must still pass **`isWhitelisted`** on that same address for bounty rules, unless the app layer maps addresses first.

### Blacklist and DAO contracts

- **`isBlacklisted`**: local **`status == 255`**, else **`oldIdentity`** fallback.
- **`isDAOContract`**: local **`status == 2`**, else **`oldIdentity`** fallback.
- **`connectAccount` / `disconnectAccount` / `getWhitelistedRoot`**: link extra EOAs to a root identity; bounty code in **`InvitesV2`** still calls **`isWhitelisted`** on the raw **`_invitee`** address.

## Reverification (how it interacts with invite eligibility)

Reverification is entirely on **`IdentityV4`**; **`InvitesV2`** only observes the resulting **`isWhitelisted`** boolean.

### Schedule

- **`reverifyDaysOptions`** is a **`uint32[]`** of **day** thresholds, strictly **ascending**, set by **`setReverifyDaysOptions(uint8[])`** (non-empty, each value **`<= 255`**).
- **`authenticationPeriod()`** returns the **largest** entry (in **days**), not seconds.

### Effective step index: `authCount`

- **`shouldReverify(account, daysSinceAuth)`** reads **`reverifyDaysOptions[authCount]`** (using a view-time **`authCount`** that may be overridden for legacy accounts; see below). If **`daysSinceAuth >=`** that value, it returns **`true`** (reverification is **due**).

### Legacy accounts (`dateAuthenticated < 1772697574`)

For both **`shouldReverify`** and **`authenticateWithTimestamp`**, if **`dateAuthenticated`** is before that cutoff, the logic treats **`authCount`** as **`reverifyDaysOptions.length - 1`** (last step) so older users start aligned with the final schedule step. This is described in-contract as temporary post-upgrade behavior.

### Admin refresh: `authenticateWithTimestamp`

- Callable only by **`IDENTITY_ADMIN_ROLE`**, when not paused, and requires **`identities[account].status == 1`**.
- Computes **`daysSinceAuthentication`** from the supplied **`timestamp`** and **`dateAuthenticated`**.
- If **`shouldReverify`** is **`true`**, increments **`authCount`**, then wraps **`authCount`** back to **`0`** if it reaches **`reverifyDaysOptions.length`** (cycles through the schedule).
- Always updates **`dateAuthenticated`** to **`timestamp`** and emits **`WhitelistedAuthenticated`**.

### Effect on invite bounties

While **`shouldReverify`** would be **`true`** for the current **`block.timestamp`**, **`isWhitelisted(account)`** is **`false`** for status-**`1`** locals. Then **`canCollectBountyFor`** fails the whitelist check for that address until authentication is renewed.

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
2. Resolve **`IDENTITY`** for the same chain and, if **`IdentityV4`**, check **`isWhitelisted`**, **`shouldReverify`**, **`lastAuthenticated`**, and **`reverifyDaysOptions`** when bounties fail despite “known” users.
3. Read **`canCollectBountyFor`** and **`users`** for both parties before asserting payouts.
4. Surface exact revert strings and eligibility flags when a bounty is not available.

## Failure cases

- Self-invite or duplicate code (`join` reverts).
- Invitee not whitelisted or **`minimumClaims` / `minimumDays`** not met.
- **`IdentityV4`** reverification due: **`isWhitelisted(invitee)`** is **`false`** until **`authenticateWithTimestamp`** refreshes **`dateAuthenticated`** (even if **`status == 1`** in storage).
- **`bountyPaid`** already true or **`bountyAtJoin`** zero.
- Contract **`active == false`** or wrong chain relative to identity.
