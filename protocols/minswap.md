# Minswap

`category:` DEX/AMM
`last_verified:` 2026-07-22
`source:` Minswap docs `https://docs.minswap.org/`; Minswap APIs `https://docs.minswap.org/developer/minswap-apis`; Minswap SDK repo `https://github.com/minswap/sdk`; AdaStat policy pages linked below
`maintained_by:` community — unverified, needs Minswap's own agent/team to confirm and correct

## Mechanism

Constant-product AMM (Uniswap-v2-style pricing) with pooled liquidity per trading pair, plus a batcher layer so users don't spend the pool UTXO directly. Later versions have iterated on the AMM design (e.g. stableswap-style curves for correlated pairs) — check current docs for which pool types are live before assuming plain constant-product applies to every pair.

## Deployments and tooling

| Mainnet | Preprod | Preview | Skills | MCP | SDK/API | System param JSON |
|---|---|---|---|---|---|---|
| [`minswap.org`](https://minswap.org/) | [`testnet-preprod.minswap.org`](https://testnet-preprod.minswap.org/) — observed no iUSD | Not found | [`github.com/minswap/skills`](https://github.com/minswap/skills) | Not found | [`github.com/minswap/sdk`](https://github.com/minswap/sdk), [`Minswap APIs`](https://docs.minswap.org/developer/minswap-apis). SDK can deposit but farm staking function not identified; API does not expose farm functionality. | Not found |

## Token standards used

- LP tokens: native asset minted per pool, one unit per pool share. Historically a bare unit (no CIP-68 metadata) — confirm current minting policy before depending on this.

## On-chain identifiers

| Role | Policy / address | Asset name hex | Asset name text | Notes |
|---|---|---|---|---|
| V1 LP token policy | [`13aa2accf2e1561723aa26871e071fdf32c867cff7e7d50ad470d62f`](https://adastat.net/policies/13aa2accf2e1561723aa26871e071fdf32c867cff7e7d50ad470d62f) | pool-specific suffix | pool-specific | Common V1 LP policy seen by indexers; verify against SDK/source before treating as exhaustive. |
| V2 LP token policy | [`f5808c2c990d86da54bfc97d89cee6efa20cd8461616359478d96b4`](https://adastat.net/policies/f5808c2c990d86da54bfc97d89cee6efa20cd8461616359478d96b4) | pool-specific suffix | pool-specific | Common V2 LP policy seen by indexers; verify against SDK/source before treating as exhaustive. |

## Concurrency approach

Batcher / order-UTXO model (see `cardano/concurrency-patterns.md`): swap/deposit/withdraw requests are submitted as order UTXOs at a known order-script address; an off-chain batcher aggregates pending orders and settles them against the pool in a single transaction on a periodic cadence.

## Composability surface

- LP tokens are transferable native assets and have been used as collateral by third-party lending protocols in the ecosystem — confirm current acceptance with the specific lending protocol, not assumed.
- Datum schema for pools/orders is protocol-defined; do not hard-code a layout without checking Minswap's current published schema/SDK, since AMM versions have changed pool datum shape across releases.

## SDK/API

- Official/public API docs: `https://docs.minswap.org/developer/minswap-apis`; base URL documented there as `https://api-mainnet-prod.minswap.org`.
- TypeScript SDK: `@minswap/sdk`, repo `https://github.com/minswap/sdk`. The SDK supports pool data, pricing, price impact, order building, and adapters for Blockfrost, Maestro, and Minswap syncer data.
- SDK gotcha: the SDK is ESM and depends on the SpaceBudz Lucid package; read its install notes before using it in a CommonJS or locked-down agent runtime.

## Gotchas

- Order fill is not synchronous with order submission — an order UTXO can sit unfilled until the next batcher run; track order-UTXO consumption, not just tx confirmation, to know a swap actually executed.
- Multiple AMM versions/pool types may coexist on-chain; a pool address/datum shape from one version won't decode with another version's expectations.
