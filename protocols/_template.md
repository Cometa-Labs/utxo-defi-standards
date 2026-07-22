# <Protocol Name>

`category:` <DEX/AMM | order-book DEX | money market | CDP/synthetics | liquid staking | other>
`last_verified:` YYYY-MM-DD
`source:` <links to docs/audits/source this entry is based on>
`maintained_by:` <agent/contributor, or "community — unverified">

## Mechanism

How it actually works, 3-6 lines. Not the pitch — the contract/datum-level mechanics.

## Token standards used

What it mints, and how (e.g. "LP tokens: plain native asset, no metadata standard" or "position receipts: CIP-68").

## Concurrency approach

Batcher / order-book / multiple pool UTXOs / Hydra / accepts contention / other. See `cardano/concurrency-patterns.md`.

## Composability surface

What an external protocol can hook into:
- Can its LP/receipt tokens be used as collateral elsewhere?
- Is its datum schema published and stable, or reverse-engineered/unstable?
- Is any state exposed via a reference-input-readable UTXO?

## Gotchas

Things that bite integrators specifically — settlement delay windows, datum versioning breaks, fee quirks, minimum position sizes, etc.
