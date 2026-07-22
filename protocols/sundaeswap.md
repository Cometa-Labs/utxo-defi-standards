# SundaeSwap

`category:` DEX/AMM
`last_verified:` 2026-07-22
`source:` SundaeSwap SDK docs `https://sundaeswap-finance.github.io/sundae-sdk/`; SundaeSwap GitHub `https://github.com/SundaeSwap-finance`; AdaStat policy pages linked below
`maintained_by:` community — unverified, needs SundaeSwap's own agent/team to confirm and correct

## Mechanism

AMM DEX; early versions used a batcher/order model similar to Minswap. Later protocol versions (referred to in ecosystem discussion as v3) moved toward reducing batcher dependency and reworking settlement — confirm which version's mechanics apply before integrating, since the concurrency and datum model differ meaningfully between versions.

## Deployments and tooling

| Mainnet | Preprod | Preview | Skills | MCP | SDK/API | System param JSON |
|---|---|---|---|---|---|---|
| [`app.sundae.fi`](https://app.sundae.fi/) | Not found | [`testnet.sundaeswap.finance`](https://testnet.sundaeswap.finance/#/swap) — has iUSD, but LP path observed missing | Not found | Not found | [`github.com/SundaeSwap-finance/sundae-sdk`](https://github.com/SundaeSwap-finance/sundae-sdk), [`SDK docs`](https://sundaeswap-finance.github.io/sundae-sdk/) | Not found |

## Token standards used

- LP tokens: native asset per pool. Exact metadata conventions (CIP-68 or bare) should be confirmed against the currently deployed version.

## On-chain identifiers

| Role | Policy / address | Asset name hex | Asset name text | Notes |
|---|---|---|---|---|
| V3 LP position token policy | [`4de79a0c17180030bff4c36825cb6e99caa007bc632f789561a26d56`](https://adastat.net/policies/4de79a0c17180030bff4c36825cb6e99caa007bc632f789561a26d56) | position-specific | position-specific | Verify current position-token layout in SundaeSwap SDK/contracts before using as an ABI. |

## Concurrency approach

Batcher / order-UTXO model in earlier versions (see `cardano/concurrency-patterns.md`). Later versions have reworked this — do not assume the v1/v2 batcher model applies to a v3 deployment without checking current docs.

## Composability surface

- LP tokens are transferable native assets; third-party acceptance as collateral is protocol-specific and should be confirmed with the accepting protocol.
- Datum/redeemer schema differs across protocol versions — version-check before hard-coding a decoder.

## SDK/API

- Official TypeScript SDK docs: `https://sundaeswap-finance.github.io/sundae-sdk/`.
- SDK source: `https://github.com/SundaeSwap-finance/sundae-sdk`; protocol/contracts org: `https://github.com/SundaeSwap-finance`.
- The SDK exists specifically to keep transaction-building datums/redeemers conformant with protocol versions; use it for write-path integrations rather than hand-encoding order datums.

## Gotchas

- Do not assume feature parity or identical settlement latency across SundaeSwap versions — verify which contract version a given pool/order address belongs to.
- As with any batcher-based version, order submission and order fill are separate events.
