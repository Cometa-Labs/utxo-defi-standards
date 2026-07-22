# Concurrency Patterns

`last_verified: 2026-07-21`

Assumes [eutxo-model.md](eutxo-model.md). The problem: a naive "shared pool UTXO" design (one UTXO holding pool reserves, spent and recreated on every swap) means only one swap per pool can land per block — every other simultaneous swap attempt against that same UTXO fails. Every production Cardano DeFi protocol works around this. There is no single standard workaround; an integrator needs to know which one a given protocol uses.

## Batcher / order-book model

Used by: most Cardano AMM DEXs (SundaeSwap, Minswap, WingRiders in their original designs).

- Users don't touch the pool UTXO directly. A user submits an **order UTXO** (locked by an order-script, datum = swap/deposit/withdraw intent) to a well-known order address.
- An off-chain (permissioned or semi-permissionless, protocol-specific) **batcher** process periodically collects pending order UTXOs, computes a single settlement, and submits one transaction that spends the pool UTXO once plus all the collected orders, producing a new pool UTXO and paying out each order.
- Effect: many users' intents get resolved in one on-chain transaction against one pool UTXO spend, sidestepping contention. Cost: added latency (users wait for the next batcher run, typically on the order of tens of seconds, protocol-dependent) and a trust/liveness dependency on the batcher.
- Integration implication: an order UTXO is not filled atomically with submission. Anything reading "did my swap happen" needs to watch for the order UTXO being consumed, not just for the submit tx to land.

## Multiple pool UTXOs / sharding

- Some designs split liquidity across several UTXOs for the same pool to allow more parallel throughput, trading off worse price execution per shard (each shard has its own smaller reserve) for more concurrent capacity. Whether a given protocol does this is protocol-specific — check its entry under `protocols/`.

## Reference inputs for shared read-only state

- Since the Vasil-era CIPs (CIP-31 reference inputs, CIP-32 inline datums, CIP-33 reference scripts), a UTXO can be **read** by many transactions in the same block without being spent, via reference inputs.
- This is the standard pattern for oracle/price-feed consumption: the oracle UTXO holds current price in its datum; consuming protocols include it as a reference input (read-only) rather than a spent input, so an unlimited number of transactions can read the same price in the same block with zero contention.
- This only works for read-only state. Anything that needs to be atomically updated (a pool's reserves after a swap) still has to be spent, and is still subject to single-consumer-per-block limits — reference inputs solve the oracle-read problem, not the shared-pool-mutation problem.

## Hydra (state channels / L2)

- Hydra "heads" let a fixed set of participants transact off-chain at near-zero latency/cost, with on-chain commitment only at head open/close (and in dispute/fraud-proof scenarios).
- Fits use cases with a known, relatively static participant set (e.g. a DEX operator + market makers) rather than open permissionless retail flow, since opening a head requires all participants to commit collateral on L1 first.
- As of this writing, Hydra is in production use by a handful of protocols/experiments, not the default mechanism for retail-facing AMMs — check a specific protocol's entry rather than assuming Hydra is involved.

## What this means for integrators

- Before assuming "my transaction either lands or fails, synchronously," check whether the protocol you're integrating uses a batcher. If it does, "order accepted" and "order filled" are different events with a gap in between, and your agent/UI needs to track both.
- Before reading protocol state, check whether it's exposed via a reference-input-readable UTXO (cheap, no contention) or requires you to independently index recent transactions to reconstruct current state (more work, more staleness risk).
