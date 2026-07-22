# Liqwid

`category:` money market
`last_verified:` 2026-07-22
`source:` Liqwid docs `https://liqwid-labs.gitbook.io/liqwid-docs`; Liqwid API docs `https://liqwid-labs.gitbook.io/liqwid-docs/api-documentation`; Liqwid qTokens docs `https://liqwid-labs.gitbook.io/liqwid-docs/faq`; AdaStat policy pages linked below
`maintained_by:` community — unverified, needs Liqwid's own agent/team to confirm and correct

## Mechanism

Pooled money-market lending (Compound-style): depositors supply an asset to a shared pool and receive a receipt token representing their claim + accrued interest; borrowers post over-collateralized deposits in one or more accepted assets and draw against them, subject to a protocol-defined collateral factor per asset and liquidation if health falls below threshold.

## Deployments and tooling

| Mainnet | Preprod | Preview | Skills | MCP | SDK/API | System param JSON |
|---|---|---|---|---|---|---|
| [`app.liqwid.finance`](https://app.liqwid.finance/) — uses a batch process; settlement timing unverified | Not available | [`dev.liqwid.finance`](https://dev.liqwid.finance/) — observed failure | Not found | Not found | [`v2 GraphQL API docs`](https://liqwid-labs.gitbook.io/liqwid-docs/api-documentation); mainnet `https://v2.api.liqwid.finance/graphql`, preview `https://v2.api.preview.liqwid.dev/graphql` | Not found |

## Token standards used

- Deposit receipts (commonly referred to in ecosystem discussion as "qTokens"): native assets representing a claim on the underlying pool, accruing value as interest accrues. Confirm current minting/metadata convention against source before depending on exact behavior.

## On-chain identifiers

| Role | Policy / address | Asset name hex | Asset name text | Notes |
|---|---|---|---|---|
| qToken policy | [`da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24`](https://adastat.net/policies/da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24) | market-specific | market-specific | Shared qToken policy seen by indexers; confirm current market units through Liqwid docs/API before valuing positions. |
| qADA | [`da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24`](https://adastat.net/policies/da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24) | `714144410000` | `qADA` | ADA supply receipt token. |
| qiUSD | [`da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24`](https://adastat.net/policies/da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24) | `71695553440000` | `qiUSD` | iUSD supply receipt token; some indexers also special-case full unit `d15c36d6dec655677acb3318294f116ce01d8d9def3cc54cdd78909b`. |
| qDJED | [`da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24`](https://adastat.net/policies/da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24) | `7144444a454400` | `qDJED` | DJED supply receipt token. |
| qMUSd | [`da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24`](https://adastat.net/policies/da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24) | `716d5553444d` | `qMUSd` | MELD/USDM-labelled market in older indexer mappings; verify current market metadata. |
| qLPS0 exact unit | [`2dbe1daa1522e5640331909fbe7458e082fe22cbc047e3c7575fcc8b`](https://adastat.net/policies/2dbe1daa1522e5640331909fbe7458e082fe22cbc047e3c7575fcc8b) | `none` | `qLPS0` | Exact-unit override used by some indexers; underlying is a SundaeSwap V3 LP position. |

## Concurrency approach

Pool-per-asset design; check current docs/source for whether pool UTXOs are batched, sharded, or otherwise designed around eUTxO contention — this materially affects deposit/withdraw/borrow latency under load.

## Composability surface

- Receipt tokens are the main composability surface: whether a third-party protocol can accept them as collateral or trade them depends on the receipt token's redemption logic being decodable/stable — confirm datum schema and any transfer restrictions with source before building on it.
- Oracle dependency: borrow/liquidation logic depends on price feeds for every accepted collateral asset — confirm which oracle provider(s) back which assets, since a stale or unsupported feed blocks borrowing/liquidation for that asset (see `patterns/composability.md`).

## SDK/API

- Official API docs: `https://liqwid-labs.gitbook.io/liqwid-docs/api-documentation`.
- Mainnet GraphQL endpoint documented by Liqwid: `https://v2.api.liqwid.finance/graphql`; preview endpoint: `https://v2.api.preview.liqwid.dev/graphql`.
- Include `X-App-Source: <app-name>` in API requests per Liqwid's integration docs.

## Gotchas

- Liquidation and collateral-factor parameters are asset-specific and change over time — do not hard-code values from any prior audit/doc without re-checking current parameters.
- Receipt token exchange rate is not 1:1 with the underlying and moves with accrued interest — don't assume face value equals redemption value.
