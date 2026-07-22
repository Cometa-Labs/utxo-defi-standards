# Dano Finance (Danogo)

`category:` CLMM DEX + money market (multi-product)
`last_verified:` 2026-07-21
`source:` reverse-engineered from on-chain transaction analysis (script hashes, redeemers, mint/burn deltas verified against real mainnet txs) — not from Dano's own docs/team. Treat mechanism claims as empirically observed behavior, not confirmed protocol design.
`maintained_by:` community — unverified, needs Dano's own agent/team to confirm and correct

## Mechanism

Two independent products under one brand:
- **CLMM** — concentrated liquidity pools (Uniswap-v3-style). Each pool is a script UTXO at the CLMM script hash; users add/remove liquidity directly against it (no batcher observed).
- **Lending** — flexible/fixed pools. Users lend ADA/tokens to a pool and receive minted receipt tokens; borrowers post collateral and draw ADA. All flexible/fixed pools share one pool-NFT policy.

## Token standards used

- CLMM LP tokens: native asset, policy = pool script hash `d8b69fc53637bcfadbc4469083f706bc293f4d9d2296646c5ca167bb` (one minting policy shared across all CLMM pools — pool identity is presumably asset-name-encoded, not policy-encoded). No CIP-68 metadata observed.
- Lending pool NFT: policy `814de8a99452972a9fa9fe2c0f59f49697f208005c001ecac1ddfd57`, shared across all flexible/fixed pools.
- Lending receipt tokens: minted by the pool validator on deposit; distinguished from returned collateral by absence from all non-collateral, non-reference tx inputs (i.e. freshly minted, not round-tripped).

## Concurrency approach

No batcher observed for either product — CLMM liquidity actions and lending deposits/borrows appear to spend/recreate the pool UTXO directly in the user's own transaction, meaning concurrent actions on the same pool are expected to contend for the same UTXO (see `cardano/concurrency-patterns.md`).

## Composability surface

- CLMM protocol (config) script hash `fa991bc2f9c4206e72d713bc3487a72e7901057cabb8d364bebeef8f` is a separate script from the LP/pool script — likely a shared reference-input config, but this hasn't been confirmed against Dano's source.
- No public API for pool APY/TVL has been found; the only known off-chain endpoint (`danogo-lending.tekoapis.com/api/v1/get-available-loan-offers`) returns *available* loan offers, not live pool rates or per-position data — insufficient for yield-tracking integrations as of this writing.
- Datum schema is unknown/unpublished — all detection above is from redeemer script hashes and token deltas, not datum decoding. Do not build a datum-dependent integration on this entry without decoding it yourself first.

## Gotchas

- Blockfrost's `/txs/{hash}/mints` endpoint returns HTTP 400 for Dano's Plutus-script mints — `tx.mint` comes back empty. Any integration relying on Blockfrost for mint detection needs a fallback (redeemer script-hash match, or UTXO-level token delta) — this is a Blockfrost-API quirk, not Dano-specific, but it bites this protocol hard because CLMM detection has no other reliable signal.
- If a user's wallet holds iUSD (Indigo synthetic, asset name hex `69555344`) it rides along as a passenger token in Dano txs and can cause naive multi-protocol detectors to also flag the tx as an unrelated Indigo interaction — don't assume single-protocol-per-tx.
- Lending action classification by net ADA flow must exclude the Plutus collateral input/return (Cardano-standard for any script tx) before computing deltas, or amounts will be inflated.
