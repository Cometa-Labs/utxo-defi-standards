# Dano Finance (Danogo)

`category:` CLMM DEX + money market (multi-product)
`last_verified:` 2026-07-22
`source:` Danogo CLMM integration docs `https://docs.danogo.io/developers/integration/concentrated-liquidity-pool-integration`; DanogoJS SDK `https://www.npmjs.com/package/@danogo-js/sdk`; AdaStat policy pages linked below; reverse-engineered on-chain analysis for identifiers not yet found in official docs
`maintained_by:` community — unverified, needs Dano's own agent/team to confirm and correct

## Mechanism

Two independent products under one brand:
- **CLMM** — concentrated liquidity pools (Uniswap-v3-style). Each pool is a script UTXO at the CLMM script hash; users add/remove liquidity directly against it (no batcher observed).
- **Lending** — flexible/fixed pools. Users lend ADA/tokens to a pool and receive minted receipt tokens; borrowers post collateral and draw ADA. All flexible/fixed pools share one pool-NFT policy.

## Deployments and tooling

| Mainnet | Preprod | Preview | Skills | MCP | SDK/API | System param JSON |
|---|---|---|---|---|---|---|
| [`dano.finance`](https://dano.finance/) | Not found | Not found | Not found | Not found | [`@danogo-js/sdk`](https://www.npmjs.com/package/@danogo-js/sdk), [`CLMM integration docs`](https://docs.danogo.io/developers/integration/concentrated-liquidity-pool-integration) | Not found |

## Token standards used

- CLMM LP tokens: native asset, one minting policy shared across all CLMM pools — pool identity is presumably asset-name-encoded, not policy-encoded. No CIP-68 metadata observed.
- Lending pool NFT: shared across all flexible/fixed pools.
- Lending receipt tokens: minted by the pool validator on deposit; distinguished from returned collateral by absence from all non-collateral, non-reference tx inputs (i.e. freshly minted, not round-tripped).
- dADA is Dano's liquid-staking receipt token.

## On-chain identifiers

| Role | Policy / address | Asset name hex | Asset name text | Notes |
|---|---|---|---|---|
| CLMM LP token policy / pool script hash | [`d8b69fc53637bcfadbc4469083f706bc293f4d9d2296646c5ca167bb`](https://adastat.net/policies/d8b69fc53637bcfadbc4469083f706bc293f4d9d2296646c5ca167bb) | pool-specific | pool-specific | Shared CLMM LP policy; pool identity appears asset-name encoded. |
| CLMM protocol/config script hash | `fa991bc2f9c4206e72d713bc3487a72e7901057cabb8d364bebeef8f` | `none` | `none` | Separate from LP/pool script; likely shared config/reference script, unconfirmed against source. |
| Lending pool NFT policy | [`814de8a99452972a9fa9fe2c0f59f49697f208005c001ecac1ddfd57`](https://adastat.net/policies/814de8a99452972a9fa9fe2c0f59f49697f208005c001ecac1ddfd57) | pool-specific | pool-specific | Shared across flexible/fixed lending pools. |
| dADA receipt policy | [`94dca24a1f1fcc2ff51cd90f32f4fe9e786d861a2dbf7d27598d26e8`](https://adastat.net/policies/94dca24a1f1fcc2ff51cd90f32f4fe9e786d861a2dbf7d27598d26e8) | unknown | `dADA` | Liquid-staking receipt token. |

## Concurrency approach

No batcher observed for either product — CLMM liquidity actions and lending deposits/borrows appear to spend/recreate the pool UTXO directly in the user's own transaction, meaning concurrent actions on the same pool are expected to contend for the same UTXO (see `cardano/concurrency-patterns.md`).

## Composability surface

- CLMM protocol/config script hash is a separate script from the LP/pool script — likely a shared reference-input config, but this hasn't been confirmed against Dano's source.
- No public API for pool APY/TVL has been found; the only known off-chain endpoint (`danogo-lending.tekoapis.com/api/v1/get-available-loan-offers`) returns *available* loan offers, not live pool rates or per-position data — insufficient for yield-tracking integrations as of this writing.
- Datum schema is unknown/unpublished — all detection above is from redeemer script hashes and token deltas, not datum decoding. Do not build a datum-dependent integration on this entry without decoding it yourself first.

## SDK/API

- CLMM integration docs: `https://docs.danogo.io/developers/integration/concentrated-liquidity-pool-integration`.
- TypeScript SDK: `@danogo-js/sdk` at `https://www.npmjs.com/package/@danogo-js/sdk`.
- The SDK documentation describes provider selection across Blockfrost, Maestro, and Kupmios/Ogmios via environment variables; verify mainnet support and contract package coverage before building a write path.

## Gotchas

- Blockfrost's `/txs/{hash}/mints` endpoint returns HTTP 400 for Dano's Plutus-script mints — `tx.mint` comes back empty. Any integration relying on Blockfrost for mint detection needs a fallback (redeemer script-hash match, or UTXO-level token delta) — this is a Blockfrost-API quirk, not Dano-specific, but it bites this protocol hard because CLMM detection has no other reliable signal.
- If a user's wallet holds iUSD (Indigo synthetic, asset name hex `69555344`) it rides along as a passenger token in Dano txs and can cause naive multi-protocol detectors to also flag the tx as an unrelated Indigo interaction — don't assume single-protocol-per-tx.
- Lending action classification by net ADA flow must exclude the Plutus collateral input/return (Cardano-standard for any script tx) before computing deltas, or amounts will be inflated.
