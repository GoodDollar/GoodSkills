# Stream guide

Authoritative GoodDocs entry: [Use G$ streaming](https://docs.gooddollar.org/for-developers/developer-guides/use-gusd-streaming).

## Goal

Create, update, or delete Superfluid constant flows using the same primitives GoodDocs describes for production G$ on Celo.

## GoodDocs summary

- G$ on Celo is a **pure Superfluid SuperToken** (no wrap step for streaming in the documented setup).
- Simple streaming uses **CFAv1Forwarder** at `0xcfA132E353cB4E398080B9700609bb008eceB125` with `createFlow`, `updateFlow`, `deleteFlow`.
- Production G$ on Celo in the guide: `0x62B8B11039FcfE5AB0C56E502b1C372A3d2a9c7A`. Development token and GoodWallet dev link appear in the same guide.
- Flow-rate math, buffers, protocol fees on streamed drips, and network scope (streaming documented for Celo) are spelled out on that page.

## Two implementation styles in this repo

1. **Forwarder (matches GoodDocs):** call CFAv1Forwarder with token, sender, receiver, flowRate, userData.
2. **Host callAgreement:** encode CFA `createFlow` / `updateFlow` / `deleteFlow` and call the Superfluid host `callAgreement`; rich ABIs live in `.agents/skills/superfluid/references/contracts/ConstantFlowAgreementV1.abi.yaml` (and `Superfluid.abi.yaml` for the host).

## Required inputs

- G$ Super Token address for the environment
- CFA forwarder or Superfluid host plus CFA implementation per your stack
- `action`: create, update, delete
- `receiver`, `flowRate` where applicable
- `rpcUrl`, chain configuration, signer

## Failure handling

- Wrong network: GoodDocs positions live streaming for G$ on Celo; other chains may not support the same flow.
- Insufficient buffer: use forwarder helpers such as `getBufferAmountByFlowrate` as in GoodDocs.
- Super App registration on Celo may require deployer allowlisting per warning in GoodDocs; direct builders to Telegram links from that page when relevant.
