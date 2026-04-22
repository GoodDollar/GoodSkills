# Envio HyperSync and HyperRPC

Use this guide when the task is high-volume historical blockchain data fetch (events, blocks, txs), especially analytics and indexing workflows.

## Official docs

- HyperSync overview: [docs.envio.dev/docs/HyperSync/overview](https://docs.envio.dev/docs/HyperSync/overview)
- HyperRPC overview: [docs.envio.dev/docs/HyperRPC/overview-hyperrpc](https://docs.envio.dev/docs/HyperRPC/overview-hyperrpc)
- HyperRPC supported networks: [docs.envio.dev/docs/HyperRPC/hyperrpc-supported-networks](https://docs.envio.dev/docs/HyperRPC/hyperrpc-supported-networks)

## What to use

- **HyperSync**: preferred for new data pipelines and heavy historical scans.
- **HyperRPC**: read-only JSON-RPC drop-in for existing RPC code paths.

## Decision rule

1. If the codebase already expects standard JSON-RPC calls, use HyperRPC first.
2. If performance is critical or the task is data-engineering style, use HyperSync.
3. For write operations (sending tx), use normal RPC providers; HyperRPC is read-only.

## GoodDollar-relevant network coverage

- Celo and XDC are supported on HyperRPC.
- Fuse is not currently listed; treat this as non-blocking and use existing providers for Fuse.

## Access and auth

- HyperRPC/HyperSync usage is account-based.
- HyperRPC requires an API key for reliable production use.
- Requests without API token are rate-limited and should be treated as non-production fallback only.
- Add API key in endpoint URL as documented by Envio.
- HyperRPC token pattern example from docs: `https://<chain>.rpc.hypersync.xyz/<api-token>`

## Practical use in this repo

- Keep subgraphs as first option for indexed protocol entities.
- Use HyperSync/HyperRPC when subgraph coverage is missing, stale, or insufficient for bulk historical pulls.
- Keep contract truth and addresses from GoodProtocol, deployment.json, and GoodDocs.
- For implementation details (client setup, query structure, supported methods), follow the Envio docs links above directly.
