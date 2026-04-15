# How UBI is minted

This document aligns agent explanations with [How GoodDollar works](https://docs.gooddollar.org/how-gooddollar-works) and [Core contracts](https://docs.gooddollar.org/for-developers/core-contracts).

## Monetary creation (protocol level)

- New G$ is created in connection with reserve mechanics: purchases into the reserve and reserve-side parameters (including reserve ratio) influence how much G$ can be issued while maintaining backing (see sustainability and issuance sections in GoodDocs).
- Selling G$ back to the reserve burns supply in that model.
- G$ that the protocol creates is allocated across UBI, savings incentives, treasury, and ecosystem uses per the distribution section of GoodDocs.

## Claim vs reserve minting (important distinction)

- **Reserve path:** Buying G$ through the reserve-backed AMM (or related Mento rails) is where mint and burn tied to the reserve model most directly apply at the token level.
- **Daily UBI `claim()`:** On `UBISchemeV2`, a successful claim typically **transfers G$ from the scheme contract’s balance** to the user (`token.transfer` in `_transferTokens`). The scheme may be **refilled** from the DAO avatar via internal `_withdrawFromDao` when configured, not necessarily minting in the same transaction as `claim()`. So describe user-facing UBI as **receipt from the UBI scheme balance**; reserve **minting** is the macro story, **transfer** is the usual claim-time mechanism.

## What claimers experience

- Verified users receive daily UBI from a pool split among those who claim in each period (GoodDocs).
- The user-facing transaction is a UBIScheme-style `claim` on chains where it is deployed; ABI and version follow your deployment.

## On-chain components (typical)

- Identity system for verification and whitelist roots.
- UBIScheme (or successor) for entitlement and claim execution.
- G$ token: value reaches users via **transfer** from scheme balance and/or broader minting economics from the reserve side depending on which action you analyze.

## Agent guidance

- Use GoodDocs for macro issuance and allocation; use UBIScheme + token transfer behavior for **claim** explanations.
- Use [deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json) for contract addresses instead of guessing.
