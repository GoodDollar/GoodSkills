# Swap guide

Use for buying or selling G$ through Mento-connected contracts on networks where they are deployed (see [Core contracts](https://docs.gooddollar.org/for-developers/core-contracts): MentoBroker, MentoReserve, ExchangeProvider, ExpansionController).

## GoodDocs alignment

- Reserve and buy or sell mechanics at the protocol level: [How GoodDollar works](https://docs.gooddollar.org/how-gooddollar-works) and [Buy and Sell G$ user guide](https://docs.gooddollar.org/user-guides) (includes reserve AMM narrative; older explorer step-by-step for Ethereum testnets remains in that page for reference).
- Integration patterns and decimals: [How to integrate the G$ token](https://docs.gooddollar.org/for-developers/developer-guides/how-to-integrate-the-gusd-token).

## Goal

Execute bounded swaps using broker quotes and correct allowances.

## Required inputs

- `direction` as buy or sell
- broker and exchange identifiers for the deployment
- amounts in correct token decimals for the chain
- `rpcUrl`, chain configuration, signer

## Execution flow

1. Confirm Mento Broker (and related) addresses for the chain from GoodDocs or deployment.json.
2. Fetch quote (`getAmountOut` or `getAmountIn` depending on direction and ABI).
3. Apply slippage bounds.
4. Approve the spent token for the broker when required.
5. Call `swapIn` or `swapOut` per your integration.
6. Return tx hash and effective amounts.

## Failure handling

- No deployment on chain: direct the user to supported networks in core contracts.
- Stale quote or tight slippage: refresh quote or relax bounds with user consent.
- Allowance or balance shortfall: report exact delta.
