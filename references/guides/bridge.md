# Bridge guide

Use for moving G$ across supported networks with deployment-specific bridge contracts.

Primary local ABI reference for MessagePassingBridge flow:

- `references/contracts/MessagePassingBridge.abi.yaml`

## GoodDocs alignment

- User flow and high-level behavior: [Bridge GoodDollars](https://docs.gooddollar.org/user-guides/bridge-gooddollars).
- Resolve supported bridge contracts per chain from [Core contracts](https://docs.gooddollar.org/for-developers/core-contracts) and [deployment.json](https://github.com/GoodDollar/GoodProtocol/blob/master/releases/deployment.json).

## Goal

Bridge with deterministic pre-checks: bridge support, allowance, amount, fee.

## Required inputs

- source and destination chain metadata
- bridge contract address for source chain
- source G$ token address
- amount in source token decimals
- signer and rpc url

## Execution flow

1. Resolve source bridge and token addresses for the network pair.
2. Run bridge eligibility checks for sender and amount via `canBridge(from, amount)`.
3. Read allowance and approve bridge spender when required.
4. Resolve transport mode (`LZ` or `AXELAR`) and estimate required native fee.
5. Send bridge transaction with nonzero `msg.value` and explicit transport method.
6. Return tx hash and normalized bridge parameters.

## Deterministic snippet

```js
import { ethers } from "ethers";

const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
const signer = new ethers.Wallet(process.env.PRIVATE_KEY, provider);

const token = new ethers.Contract(
  process.env.GOODDOLLAR_ADDRESS,
  [
    "function allowance(address,address) view returns (uint256)",
    "function approve(address,uint256) returns (bool)",
  ],
  signer,
);

const bridge = new ethers.Contract(
  process.env.BRIDGE_ADDRESS,
  [
    "function canBridge(address,uint256) view returns (bool,string)",
    "function toLzChainId(uint256) view returns (uint16)",
    "function estimateSendFee(uint16,address,address,uint256,bool,bytes) view returns (uint256,uint256)",
    "function bridgeToWithLz(address,uint256,uint256,bytes) payable",
    "function bridgeToWithAxelar(address,uint256,uint256,address) payable",
  ],
  signer,
);

const owner = await signer.getAddress();
const targetChainId = Number(process.env.TARGET_CHAIN_ID);
const recipient = process.env.RECIPIENT;
const amount = ethers.parseUnits(process.env.AMOUNT, Number(process.env.DECIMALS));
const transport = (process.env.BRIDGE_TRANSPORT || "LZ").toUpperCase();

const [canBridge, reason] = await bridge.canBridge(owner, amount);
if (!canBridge) throw new Error(`Bridge blocked: ${reason}`);

const allowance = await token.allowance(owner, process.env.BRIDGE_ADDRESS);
if (allowance < amount) {
  const approveTx = await token.approve(process.env.BRIDGE_ADDRESS, amount);
  await approveTx.wait();
}

let tx;

if (transport === "LZ") {
  const dstEid = await bridge.toLzChainId(targetChainId);
  if (dstEid === 0) throw new Error("Unsupported target chain for LayerZero");

  const adapterParams = process.env.LZ_ADAPTER_PARAMS || "0x";
  const [nativeFee] = await bridge.estimateSendFee(dstEid, owner, recipient, amount, false, adapterParams);
  if (nativeFee <= 0n) throw new Error("Estimated LayerZero fee is zero");

  tx = await bridge.bridgeToWithLz(recipient, targetChainId, amount, adapterParams, {
    value: nativeFee,
  });
} else if (transport === "AXELAR") {
  const nativeFee = ethers.parseEther(process.env.AXELAR_FEE_ETH || "0.01");
  tx = await bridge.bridgeToWithAxelar(recipient, targetChainId, amount, owner, {
    value: nativeFee,
  });
} else {
  throw new Error("Unsupported BRIDGE_TRANSPORT. Use LZ or AXELAR");
}

const receipt = await tx.wait();
console.log(
  JSON.stringify(
    {
      txHash: receipt.hash,
      sourceBridge: process.env.BRIDGE_ADDRESS,
      targetChainId,
      transport,
      recipient,
      amount: amount.toString(),
    },
    null,
    2,
  ),
);
```

## Failure handling

- unsupported destination: return targetChainId, bridge address, and transport mode
- fee too low (`LZ_FEE` or underpriced Axelar fee): re-estimate and retry with user confirmation
- approval or balance issue: return required delta
