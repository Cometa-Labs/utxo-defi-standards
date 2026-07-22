# Indigo Protocol

`category:` CDP / synthetics
`last_verified:` 2026-07-21
`source:` public docs — not verified against Indigo's own source/team; treat specifics below as a starting point, not ground truth
`maintained_by:` community — unverified, needs Indigo's own agent/team to confirm and correct

## Mechanism

CDP-based synthetic asset issuance: a user locks collateral (ADA and/or other accepted assets, protocol-dependent) into a CDP-style position and mints synthetic assets ("iAssets") tracking the price of a real-world reference (e.g. a synthetic tracking a fiat currency or other reference asset), subject to a minimum collateralization ratio and liquidation if the ratio falls below threshold. Conceptually parallel to Synthetix-style synthetics, adapted to eUTxO (each CDP is its own UTXO+datum rather than a shared contract storage slot per user).

## Token standards used

- iAssets: native assets minted/burned by the CDP validator as positions open/close/adjust. Confirm current metadata convention (bare vs. CIP-68) against source.

## Concurrency approach

Each CDP is its own UTXO (one per position), which sidesteps pool-style contention for position management — but price-feed reads for minting/liquidation checks still depend on oracle availability (see `patterns/composability.md`). Confirm with source whether stability-pool or global protocol-level actions (e.g. system-wide liquidation triggers) introduce any shared-UTXO contention points.

## Composability surface

- iAssets are transferable native assets and can in principle be used elsewhere (DEX pairs, collateral) — actual third-party acceptance is protocol-specific and should be confirmed per accepting protocol.
- CDP datum schema is the main integration surface for anything reading position health programmatically — confirm current schema/version with source rather than assuming stability across protocol upgrades.

## Gotchas

- Minimum collateralization ratio and accepted collateral types are asset-specific and change over time — don't hard-code prior values.
- Liquidation depends on oracle freshness for the referenced price; a stale oracle read has different failure implications here than in a simple swap (see `patterns/composability.md` on oracle staleness).
