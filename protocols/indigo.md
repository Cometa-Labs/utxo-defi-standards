# Indigo Protocol

`category:` CDP / synthetics
`last_verified:` 2026-07-22
`source:` Indigo docs `https://docs.indigoprotocol.io/readme/about-indigo`; Indigo SDK package `https://www.npmjs.com/package/@indigoprotocol/indigo-sdk`; Indigo MCP `https://github.com/IndigoProtocol/indigo-mcp`; system params linked below; AdaStat address pages linked below
`maintained_by:` community — unverified, needs Indigo's own agent/team to confirm and correct

## Mechanism

CDP-based synthetic asset issuance: a user locks collateral (ADA and/or other accepted assets, protocol-dependent) into a CDP-style position and mints synthetic assets ("iAssets") tracking the price of a real-world reference (e.g. a synthetic tracking a fiat currency or other reference asset), subject to a minimum collateralization ratio and liquidation if the ratio falls below threshold. Conceptually parallel to Synthetix-style synthetics, adapted to eUTxO (each CDP is its own UTXO+datum rather than a shared contract storage slot per user).

## Deployments and tooling

| Mainnet | Preprod | Preview | Skills | MCP | SDK/API | System param JSON |
|---|---|---|---|---|---|---|
| [`app.indigoprotocol.io/dashboard`](https://app.indigoprotocol.io/dashboard) | [`preprod.indigoprotocol.io`](https://preprod.indigoprotocol.io/) | [`preview.indigoprotocol.io`](https://preview.indigoprotocol.io/) — observed wallet-connect failure | [`indigo-ai`](https://github.com/IndigoProtocol/indigo-ai.git) | [`indigo-mcp`](https://github.com/IndigoProtocol/indigo-mcp), [`cardano-mcp`](https://github.com/IndigoProtocol/cardano-mcp) | [`@indigoprotocol/indigo-sdk`](https://www.npmjs.com/package/@indigoprotocol/indigo-sdk) / [`github.com/IndigoProtocol/indigo-sdk`](https://github.com/IndigoProtocol/indigo-sdk). Use this package rather than `@indigo-labs/indigo-sdk`. | [`mainnet v21 lrp`](https://config.indigoprotocol.io/mainnet/mainnet-system-params-v21-lrp.json), [`mainnet v2 ctl92`](https://config.indigoprotocol.io/mainnet/mainnet-system-params-v2-ctl92.json), [`testnet v21`](https://config.indigoprotocol.io/testnet/testnet-system-params-v21.json), [`sdk test fixture`](https://github.com/IndigoProtocol/sdk/blob/ee37c9655dfde9b6c8215fcfa26c1d1704845234/tests/data/system-params.json#L4) |

## Token standards used

- iAssets: native assets minted/burned by the CDP validator as positions open/close/adjust. Confirm current metadata convention (bare vs. CIP-68) against source.
- Main iAssets documented by Indigo include iUSD, iBTC, and iETH; exact policy IDs and metadata conventions should be read from Indigo's current source/API before hard-coding.

## Concurrency approach

Each CDP is its own UTXO (one per position), which sidesteps pool-style contention for position management — but price-feed reads for minting/liquidation checks still depend on oracle availability (see `patterns/composability.md`). Confirm with source whether stability-pool or global protocol-level actions (e.g. system-wide liquidation triggers) introduce any shared-UTXO contention points.

## Composability surface

- iAssets are transferable native assets and can in principle be used elsewhere (DEX pairs, collateral) — actual third-party acceptance is protocol-specific and should be confirmed per accepting protocol.
- CDP datum schema is the main integration surface for anything reading position health programmatically — confirm current schema/version with source rather than assuming stability across protocol upgrades.

## On-chain identifiers

| Role | Policy / address | Asset name hex | Asset name text | Notes |
|---|---|---|---|---|
| iUSD native asset | [`f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880`](https://adastat.net/policies/f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880) | `69555344` | `iUSD` | Indigo synthetic asset. |
| iBTC native asset | [`f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880`](https://adastat.net/policies/f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880) | `69425443` | `iBTC` | Indigo synthetic asset. |
| iETH native asset | [`f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880`](https://adastat.net/policies/f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880) | `69455448` | `iETH` | Indigo synthetic asset. |
| INDY native asset | [`533bb94a8850ee3ccbe483106489399112b74c905342cb1792a797a0`](https://adastat.net/policies/533bb94a8850ee3ccbe483106489399112b74c905342cb1792a797a0) | `494e4459` | `INDY` | Governance/staking asset. |

| Role | Address |
|---|---|
| CDP validator | [`addr1w8qmxkacjdffxah0l3qg8hq2pmvs58q8lcy42zy9kda2ylc6dy5r4`](https://adastat.net/addresses/addr1w8qmxkacjdffxah0l3qg8hq2pmvs58q8lcy42zy9kda2ylc6dy5r4) |
| CDP validator | [`addr1wywp3d3q2l3x37k38fmqzm83xlhsf83aelszr4kk2xwqh65sl0wl5`](https://adastat.net/addresses/addr1wywp3d3q2l3x37k38fmqzm83xlhsf83aelszr4kk2xwqh65sl0wl5) |
| Staking validator | [`addr1w9z8k3czq3zge2v8vam0ypkqzm03w5t3vx4f8ey5mf7eln8dqpss`](https://adastat.net/addresses/addr1w9z8k3czq3zge2v8vam0ypkqzm03w5t3vx4f8ey5mf7eln8dqpss) |

## SDK/API/MCP

- SDK package: `@indigoprotocol/indigo-sdk` (`^1.0.6` observed in integration notes) at `https://www.npmjs.com/package/@indigoprotocol/indigo-sdk`; repository `https://github.com/IndigoProtocol/indigo-sdk`.
- MCP package: `@indigoprotocol/indigo-mcp` / `https://github.com/IndigoProtocol/indigo-mcp`. It documents stdio/http transport and Blockfrost/indexer environment variables.
- SDK appears to require `system-params.json`; mainnet JSON is published under `config.indigoprotocol.io`, but testnet linkage is not clearly documented and should be verified before using preprod/preview write paths.
- SDK/MCP packages are useful for agent and read/write integrations, but liquidation safety still depends on live oracle prices and current governance parameters.

## Gotchas

- Minimum collateralization ratio and accepted collateral types are asset-specific and change over time — don't hard-code prior values.
- Liquidation depends on oracle freshness for the referenced price; a stale oracle read has different failure implications here than in a simple swap (see `patterns/composability.md` on oracle staleness).
