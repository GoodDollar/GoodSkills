# Faucet and test flows

The on-chain **Faucet** in GoodProtocol tops up user wallets (native gas on Fuse-style networks) subject to **identity**, **rate limits**, and **admin/relayer** roles. GoodDocs lists **Faucet** under [Core contracts](https://docs.gooddollar.org/for-developers/core-contracts); implementation: [`contracts/fuseFaucet/Faucet.sol`](https://github.com/GoodDollar/GoodProtocol/blob/master/contracts/fuseFaucet/Faucet.sol).

## Purpose

- Whitelisted users (or their **connected** roots via **Identity.getWhitelistedRoot**) can receive **small native balance top-ups** so they can pay gas for claims and other actions.
- **RELAYER_ROLE** can top **non-whitelisted** targets where the contract allows (see **`onlyAuthorized`**).

## Main entrypoints

- **`topWallet(address payable user)`** — applies **`onlyAuthorized`**, **`toppingLimit`**, and **`reimburseGas`** (refunds caller gas at configured **`gasPrice`**). Internally **`_topWallet`** computes **`getToppingAmount`**, requires meaningful **`toTop` vs minTopping`**, updates **daily/weekly** accounting, **`transfer(toTop)`** to the **target** wallet, emits **`WalletTopped`**.
- **`canTop(address user)`** — view helper for UI or agents to preflight eligibility.
- **`getToppingAmount` / `getToppingAmount(address)`** — base amount from **`gasTopping * gasPrice`**, doubled if **`votingContract`** is set and the user has voting power.

## Access and limits (high level)

- **`onlyAuthorized`**: target must have a **non-zero whitelisted root**, or caller must have **RELAYER_ROLE**; **`bans[user]`** must be expired.
- **Daily**: up to **`maxDailyToppings`** per accounting key (root or raw user when not connected).
- **Weekly**: **`weeklyToppingSum`** capped relative to **`perDayRoughLimit`**, **`gasPrice`**, **`maxPerWeekMultiplier`**.
- **New wallets**: **`maxDailyNewWallets`** and **`notFirstTime`** gate cold addresses unless whitelisted.

## Other surface

- **`onTokenTransfer`** — swaps received ERC20 (cToken-style caller) to ETH via a decoded Uniswap-like router; **no slippage guard** in contract comments; treat as **unsafe for large amounts**.
- Admin setters: **`setGasTopping`**, **`setGasPrice`**, **`setMinTopping`**, **`setVotingContract`**, **`banAddress`** (relayer), UUPS upgrade via **DEFAULT_ADMIN_ROLE**.

## Test and dev usage

- Resolve **Faucet** address from [deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json) for **Fuse / Celo / XDC** (see GoodDocs table; mainnet Ethereum may show **not deployed** for Faucet).
- For **end-user test G$** on Celo, GoodDocs streaming guide references **development** token addresses in [How to integrate the G$ token](https://docs.gooddollar.org/for-developers/developer-guides/how-to-integrate-the-gusd-token) and [Use G$ streaming](https://docs.gooddollar.org/for-developers/developer-guides/use-gusd-streaming) — that is separate from the **Faucet** contract but part of the same **test** story.

## Agent guidance

1. Confirm **chain** and **Faucet** deployment before suggesting **`topWallet`**.
2. Use **`canTop`** and **`getToppingAmount`** to explain rejections (**`max daily toppings`**, **`low toTop`**, **`not authorized`**, **`banned`**, weekly cap).
3. Do not present **`onTokenTransfer`** as a general-purpose swap path without warning about **slippage**.
