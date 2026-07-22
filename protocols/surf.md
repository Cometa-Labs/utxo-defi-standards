# Surf Lending

`category:` money market (pooled, isolated per collateral)
`last_verified:` 2026-07-21
`source:` reverse-engineered from on-chain transaction analysis (script hashes, redeemer/datum-derived NFT roles verified against real mainnet txs) — not from Surf's own docs/team. Treat mechanism claims as empirically observed behavior, not confirmed protocol design.
`maintained_by:` community — unverified, needs Surf's own agent/team to confirm and correct

## Mechanism

Isolated lending pools: each pool accepts exactly one collateral token against ADA. Lenders deposit ADA into a pool and receive a minted receipt token; borrowers post the pool's designated collateral token to draw ADA. Uses a **two-phase batcher model** distinct from a simple order-book: (1) the user submits an order UTXO to a request/order script with no script execution (plain payment tx, no redeemers); (2) a batcher later spends the order + pool UTXO together in one Plutus tx, minting receipts to the user. Both txs are the same logical user action (e.g. one `lend`), submitted as two separate on-chain transactions.

## Token standards used

- Receipt tokens: native asset, minting policy `daa59d07e4c35a2b32c36017534b3346e0576968d45d5b0e5cf41436` (pool-specific, at least for the COCK/ADA pool — confirm whether policy is shared or per-pool before generalizing). No CIP-68 metadata observed. Exchange rate vs. underlying ADA is not 1:1 and not exposed by any known API.
- Position tracking: a per-user "position token" (asset name = numeric string), policy `d2ce01940643082f59198a9d2868183a1482eadb66c629351f64d45a`, held at a companion address — functions as a per-position state marker rather than a transferable receipt.

## Concurrency approach

Batcher model (see `cardano/concurrency-patterns.md`), but split across two identity layers rather than one pool UTXO:
- `REF_POOL_NFT` (policy `26eec049c58a89d632e318ec4a102ab5a8cce86d40e2aeb29f6080f7`) — global, reference-input-only, protocol-wide params read by every Surf tx.
- `POOL_INFO_NFT` — reference-input-only, per-pool config (accepted collateral, interest rate, LTV), read but not spent by the batcher.
- `POOL_NFT` — the live, spendable pool-balance UTXO; both `POOL_INFO_NFT` and `POOL_NFT` currently share policy `b6cd0e95575ffa5d9efd08a0d06961cf6dfd806a7104ea90b34dd18` (distinguished by address/role, not policy). This is the single contention point per pool — spent and recreated on every batcher execution.
- All pool/order/reference addresses for a given pool share a common bech32 staking-credential substring, which is the protocol's shared staking script.

## Composability surface

- `POOL_INFO_NFT`'s inline datum (at its dedicated reference address) is the documented way to discover a pool's accepted collateral policy/asset-name and risk parameters (interest rate, LTV) without spending anything — genuinely reference-input-readable state, usable by external protocols.
- Receipt tokens are transferable native assets in principle, but no exchange-rate API is known, so a third party accepting them as collateral would need to derive redemption value from on-chain state directly.
- Order-script and pool-script validator hashes (`3c3bb6f0cfa3d82d7e2877f5393939c682c8626fd4a732592970ae38`, `90d57c3dec8ba7544bc6fdb309f22629083d7b76ff8329bac95fc35f`) are shared across all pools — a listener for batcher-execution txs doesn't need per-pool address lists, only the order-creation leg does (order script address is pool-specific).

## Gotchas

- The order-creation leg has zero script execution (no redeemers) and looks like an ordinary payment — do not assume "no Plutus scripts involved" means "not a protocol interaction."
- In the batcher-execution tx, the user has *no spend inputs at all* — they only appear in outputs (receipt tokens + a ~2 ADA min-UTXO carrier, which is min-ADA, not a fee). Computing the lend amount from the user's own wallet-side net flow gives zero/garbage; use the pool UTXO's ADA delta (output − input) instead.
- Receipt-token minting is Plutus-script minting, so Blockfrost's `/mints` endpoint can return HTTP 400 on the execution tx — detection must rely on redeemer script-hash matches, not `tx.mint`.
- A wallet holding iUSD (or other Indigo iAssets) as a passenger token can cause the same tx to also match Indigo's (weak) detection — resolve by priority, not by assuming exclusivity.
- New pools require discovering their `POOL_INFO_NFT` reference UTXO and decoding its datum to learn accepted collateral; shared validator hashes mean batcher-execution detection works immediately for a new pool, but order-creation detection needs the new pool's order-script address added explicitly.
