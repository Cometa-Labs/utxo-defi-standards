# Minswap

`category:` DEX/AMM
`last_verified:` 2026-07-21
`source:` public docs — not verified against Minswap's own source/team; treat specifics below as a starting point, not ground truth
`maintained_by:` community — unverified, needs Minswap's own agent/team to confirm and correct

## Mechanism

Constant-product AMM (Uniswap-v2-style pricing) with pooled liquidity per trading pair, plus a batcher layer so users don't spend the pool UTXO directly. Later versions have iterated on the AMM design (e.g. stableswap-style curves for correlated pairs) — check current docs for which pool types are live before assuming plain constant-product applies to every pair.

## Token standards used

- LP tokens: native asset minted per pool, one unit per pool share. Historically a bare unit (no CIP-68 metadata) — confirm current minting policy before depending on this.

## Concurrency approach

Batcher / order-UTXO model (see `cardano/concurrency-patterns.md`): swap/deposit/withdraw requests are submitted as order UTXOs at a known order-script address; an off-chain batcher aggregates pending orders and settles them against the pool in a single transaction on a periodic cadence.

## Composability surface

- LP tokens are transferable native assets and have been used as collateral by third-party lending protocols in the ecosystem — confirm current acceptance with the specific lending protocol, not assumed.
- Datum schema for pools/orders is protocol-defined; do not hard-code a layout without checking Minswap's current published schema/SDK, since AMM versions have changed pool datum shape across releases.

## Gotchas

- Order fill is not synchronous with order submission — an order UTXO can sit unfilled until the next batcher run; track order-UTXO consumption, not just tx confirmation, to know a swap actually executed.
- Multiple AMM versions/pool types may coexist on-chain; a pool address/datum shape from one version won't decode with another version's expectations.
