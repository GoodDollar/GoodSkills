# On- and off-ramp service via stable token swap

Use this note for the service pattern where ramp providers do not list G$ directly. The practical path is: ramp in/out with a listed stable token (for example cUSD), then swap between stable and G$ on-chain.

## Why this is required

- Most on-/off-ramp providers list mainstream stable tokens, not G$.
- Service needs a bridge asset for fiat rails.
- Stable token becomes the integration point with ramp providers, while G$ remains the in-app asset.

## On-chain source of truth

- Solidity: [`contracts/utils/BuyGDClone.sol`](https://github.com/GoodDollar/GoodProtocol/blob/master/contracts/utils/BuyGDClone.sol)
- Components:
  - `BuyGDCloneFactory`
  - `BuyGDCloneV2`
  - `DonateGDClone`
- Deployments: [releases/deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json)

## Service architecture

The factory deploys EIP-1167 minimal clones and wires swap infrastructure (router, oracle, quoter, optional Mento broker configuration, G$ token, stable token).

Each user gets a deterministic clone address from owner-based salt:

- `predict(owner)` for buy clone
- `predictDonation(owner, donor)` for donation clone

This is why clone-per-user design is used:

- predictable per-user addresses for audit and routing
- isolated execution context
- cheaper deployment than full contract instances

## Ramp flows

### On-ramp (fiat -> stable -> G$)

1. User receives listed stable token from ramp provider.
2. Service resolves user clone via `predict` (or `create` if missing).
3. Swap stable -> G$ through clone (`createAndSwap` or swap after `create`).
4. Return resulting G$ to destination wallet.

### Off-ramp (G$ -> stable -> fiat)

1. User provides G$ to service flow.
2. Service swaps G$ -> listed stable token using configured rails.
3. Stable token is sent to off-ramp provider destination for fiat payout.

## Entrypoints to use

- `create(owner)`
- `createAndSwap(owner, minAmount)`
- `predict(owner)`
- `createDonation`, `createDonationAndSwap`, `predictDonation`

## Operational requirements

1. Resolve factory address per chain from `deployment.json`.
2. Verify clone address against `predict` before routing funds.
3. Enforce slippage guard (`minAmount`) on swap paths.
4. Record chain id, factory, predicted/actual clone, and tx hashes.

## Risks

- Wrong factory or wrong chain causes permanent fund loss risk.
- Stale router/oracle/mento config can fail swap or produce bad execution.
- Missing `minAmount` protection increases slippage risk.

## Cross-reference

- User narrative: [Buy and Sell G$](https://docs.gooddollar.org/user-guides)
- Token integration details: [How to integrate the G$ token](https://docs.gooddollar.org/for-developers/developer-guides/how-to-integrate-the-gusd-token)
- Broker ABI: `references/contracts/MentoBroker.abi.yaml`
