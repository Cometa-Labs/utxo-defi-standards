# Composability Patterns

`last_verified: 2026-07-21`

Cross-protocol patterns and their failure modes. Assumes [cardano/eutxo-model.md](../cardano/eutxo-model.md) and [cardano/token-standards.md](../cardano/token-standards.md).

## The missing-shared-ABI problem

EVM composability works because ERC-20/721 give every contract a common call interface, and a contract calling another contract executes atomically in the same call stack. eUTxO has neither:

- No common interface: every protocol's datum schema and redeemer types are bespoke. There's no equivalent of "call `transfer(address,uint256)` and know it'll work on any ERC-20."
- No cross-contract calls: composing two protocols in one transaction means constructing a single transaction that spends/references UTXOs from both, satisfying both validators' constraints simultaneously — the integrator (or their tooling) has to understand both protocols' transaction-building rules, there's no "protocol A calls protocol B" primitive.

Practical consequence: composing protocol A with protocol B requires bespoke integration work against both, redone whenever either changes its datum schema. This repo exists to make that bespoke work reusable instead of rediscovered per integrator.

## LP tokens as cross-protocol collateral

- An AMM's LP token is a plain native asset (see `cardano/token-standards.md`), so nothing stops another protocol (a money market, say) from accepting it as loan collateral — it's just a token in a wallet's UTXO.
- What breaks this in practice: the accepting protocol needs a reliable price for the LP token (typically computed from the underlying pool's reserves, which requires reading the AMM's own pool UTXO/datum — a dependency on that AMM's specific schema staying stable), and needs to handle the LP token being redeemable back into the underlying pair at a rate that moves with pool activity, not a fixed rate.
- Check a lending protocol's entry for which LP tokens (if any) it accepts and how it prices them — this is protocol-specific and not all money markets support it.

## Oracle / price-feed reads

- Standard pattern post-Vasil: an oracle protocol maintains a UTXO whose inline datum holds the current price(s), updated periodically by the oracle's own update mechanism. Consuming protocols include it as a **reference input** (read, don't spend — see `cardano/concurrency-patterns.md`), so any number of consumers can read the same price in the same block without contending with each other.
- Two Cardano oracle providers commonly referenced in this space are Charli3 and Orcfax — verify current status/coverage directly with each before depending on a specific feed; oracle availability per asset pair changes.
- Failure mode for integrators: reading a stale reference input (oracle hasn't updated recently) is not a transaction failure — the tx still validates against whatever price was there. Protocols relying on oracle reads for solvency-critical logic (liquidations, CDP minting) need their own staleness checks against the oracle UTXO's last-update timestamp, not just "the read succeeded."

## Receipt/position tokens

- Money markets and staking-derivative protocols commonly mint a receipt token (e.g. a "qToken"-style asset) representing a claim on a deposited position, often via CIP-68 so the claim's terms are on-chain and machine-readable rather than implied.
- Composability here depends entirely on whether the issuing protocol treats the receipt token's transfer as equivalent to transferring the underlying claim (most do) and whether its redemption datum schema is documented/stable enough for a third protocol to parse without going through the issuing protocol's own UI/SDK.

## What tends to actually break integrations

- A protocol changes its datum schema (adds a field, reorders a constructor) without a version marker — anything that hard-decoded the old CDDL breaks silently or reads garbage. Check whether a protocol's entry notes datum versioning before building a hard dependency on its exact layout.
- Assuming synchronous settlement on a batcher-based protocol (see `cardano/concurrency-patterns.md`) — "my transaction landed" is not "my order filled."
- Treating an LP or receipt token's face value as its redemption value without re-deriving current pool state/exchange rate.
