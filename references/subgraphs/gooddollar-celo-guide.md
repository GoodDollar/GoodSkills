# GoodDollar Celo — Subgraph Usage Guide

Companion to `gooddollar-celo.graphql`.

## Endpoint

- Explorer: [GoodDollarCelo](https://thegraph.com/explorer/subgraphs/F7314rxGdcpKPC1nN5KCoFW84EGRoUyzseY2sAT9PEkw?view=Query&chain=arbitrum-one)
- Gateway form: `https://gateway.thegraph.com/api/subgraphs/id/F7314rxGdcpKPC1nN5KCoFW84EGRoUyzseY2sAT9PEkw`

---

## Entity Overview

### UBI and usage statistics

**DailyUBI** — day-level UBI pool/quota activity and cycle fields.  
**WalletStat** — wallet behavior aggregates: tx counts/values, claim stats, active/whitelist indicators.  
**TransactionStat** — day-level transaction totals and circulation view.  
**GlobalStatistics** — global claim and distribution rollups.

### Additional UBI history entities

**UBICollected** — collected UBI/community-pool values by block event.  
**UBIHistory** — timeline totals for daily UBI/community-pool.

---

## Typical Questions This Subgraph Answers

- How much UBI was distributed on a day/cycle?
- Which wallets are active or recently claiming?
- What are aggregate tx/circulation trends?
- How did collected UBI/community-pool values evolve over time?

---

## Query Discipline

- Use lowercase address strings in filters.
- Use string-safe handling for large integer values.
- Use `_meta` to validate freshness before cross-day analytics.
- Use authenticated gateway access for programmatic queries.
