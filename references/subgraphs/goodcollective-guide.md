# GoodCollective — Subgraph Usage Guide

Companion to `goodcollective.graphql`.

## Endpoint

- Explorer: [GoodCollective](https://thegraph.com/explorer/subgraphs/3LbJh9DXhJVvuVDdm5i6StNboJmL9oMNNkBaKyzc4Y8Y?view=Query&chain=arbitrum-one)
- Gateway form: `https://gateway.thegraph.com/api/subgraphs/id/3LbJh9DXhJVvuVDdm5i6StNboJmL9oMNNkBaKyzc4Y8Y`

---

## Entity Overview

### Core collective graph

**Collective** — pool identity, limits/settings links, totals, and claim/payment counters.  
**Donor** / **Steward** — participant-level donation/support state.  
**DonorCollective** / **StewardCollective** — join entities tying participants to a collective.

### Claim and support flow

**Claim** and **ClaimEvent** — reward claim lifecycle and per-claim reward events.  
**SupportEvent** — support/donation change events across donor/collective links.

### Metadata and policy entities

**IpfsCollective** — IPFS metadata projection.  
**PoolSettings**, **UBILimits**, **SafetyLimits** — pool policy and operational bounds.  
**ProvableNFT** — NFT linkage used in claim/reward flows.

---

## Typical Questions This Subgraph Answers

- Which donors/stewards are attached to a collective?
- How much was donated/rewarded per collective and per participant?
- Which claim events occurred and what reward quantities were emitted?
- What limits and settings govern a specific collective pool?

---

## Query Discipline

- Use authenticated gateway access for programmatic queries.
- Lowercase address-like identifiers where applicable.
- Validate `_meta` before operational dashboards or reporting exports.
