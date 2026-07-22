# SundaeSwap

`category:` DEX/AMM
`last_verified:` 2026-07-21
`source:` public docs — not verified against SundaeSwap's own source/team; treat specifics below as a starting point, not ground truth
`maintained_by:` community — unverified, needs SundaeSwap's own agent/team to confirm and correct

## Mechanism

AMM DEX; early versions used a batcher/order model similar to Minswap. Later protocol versions (referred to in ecosystem discussion as v3) moved toward reducing batcher dependency and reworking settlement — confirm which version's mechanics apply before integrating, since the concurrency and datum model differ meaningfully between versions.

## Token standards used

- LP tokens: native asset per pool. Exact metadata conventions (CIP-68 or bare) should be confirmed against the currently deployed version.

## Concurrency approach

Batcher / order-UTXO model in earlier versions (see `cardano/concurrency-patterns.md`). Later versions have reworked this — do not assume the v1/v2 batcher model applies to a v3 deployment without checking current docs.

## Composability surface

- LP tokens are transferable native assets; third-party acceptance as collateral is protocol-specific and should be confirmed with the accepting protocol.
- Datum/redeemer schema differs across protocol versions — version-check before hard-coding a decoder.

## Gotchas

- Do not assume feature parity or identical settlement latency across SundaeSwap versions — verify which contract version a given pool/order address belongs to.
- As with any batcher-based version, order submission and order fill are separate events.
