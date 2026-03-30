# BuyGDCloneFactory deep research

Use this note for **clone-based buy** and donation flows. Canonical user-facing context remains in [GoodDocs](https://docs.gooddollar.org/) and [Core contracts](https://docs.gooddollar.org/for-developers/core-contracts).

## Source of truth

- **Solidity:** [`contracts/utils/BuyGDClone.sol`](https://github.com/GoodDollar/GoodProtocol/blob/master/contracts/utils/BuyGDClone.sol) — defines **`BuyGDCloneFactory`**, **`BuyGDCloneV2`**, and **`DonateGDClone`** (EIP-1167 minimal clones with deterministic salts).
- **Deployment addresses:** [releases/deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json) (and NameService if wired).

## Factory role

The constructor deploys immutable **implementation** contracts for **`BuyGDCloneV2`** and **`DonateGDClone`**, wiring Uniswap-style **router**, **oracle**, **quoter**, optional **Mento** (`IBroker`, exchange provider, `exchangeId`), and fixed **G$** / **stable** token addresses. Per-clone instances are created with **`Clones.cloneDeterministic`** so addresses are predictable.

## Entrypoints (agents)

- **`create(address owner)`** — deploy clone, **`initialize(owner)`**, return clone address.
- **`createAndSwap(address owner, uint256 minAmount)`** — **`create`** then **`BuyGDCloneV2.swap(minAmount, msg.sender)`**.
- **`predict(address owner)`** — view predicted clone address for **`owner`** (salt `keccak256(abi.encode(owner))`).
- **`createDonation` / `createDonationAndSwap` / `predictDonation`** — donation variant with extra owner and calldata for **`DonateGDClone`**.

Custom errors on the factory include **`NOT_GD_TOKEN`**, **`INVALID_TWAP`**, **`RECIPIENT_ZERO`**, **`ZERO_MINAMOUNT`** (see source for where each applies).

## Agent workflow

1. Resolve factory address from `deployment.json` for the target network.
2. For a given **`owner`**, use **`predict`** when you need a deterministic address before **`create`**.
3. Record chain id, clone address, and tx hashes for any **`create`** / **`createAndSwap`** path.
4. Route user funds only through clones verified against **`predict`** or a trusted registry.

## Cross-reference

- Reserve and buy narrative: [Buy and Sell G$](https://docs.gooddollar.org/user-guides).
- G$ integration (decimals, `transferAndCall`): [How to integrate the G$ token](https://docs.gooddollar.org/for-developers/developer-guides/how-to-integrate-the-gusd-token).
- Mento broker surface: `references/contracts/MentoBroker.abi.yaml`.

## Risks

- Wrong factory or clone address or stale Mento/router config causes failed swaps or lost funds.
- Echo factory address, predicted or actual clone address, and chain id in agent outputs for auditability.
