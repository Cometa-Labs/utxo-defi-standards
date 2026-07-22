# Liqwid

`category:` money market
`last_verified:` 2026-07-21
`source:` public docs — not verified against Liqwid's own source/team; treat specifics below as a starting point, not ground truth
`maintained_by:` community — unverified, needs Liqwid's own agent/team to confirm and correct

## Mechanism

Pooled money-market lending (Compound-style): depositors supply an asset to a shared pool and receive a receipt token representing their claim + accrued interest; borrowers post over-collateralized deposits in one or more accepted assets and draw against them, subject to a protocol-defined collateral factor per asset and liquidation if health falls below threshold.

## Token standards used

- Deposit receipts (commonly referred to in ecosystem discussion as "qTokens"): native assets representing a claim on the underlying pool, accruing value as interest accrues. Confirm current minting/metadata convention against source before depending on exact behavior.

## Concurrency approach

Pool-per-asset design; check current docs/source for whether pool UTXOs are batched, sharded, or otherwise designed around eUTxO contention — this materially affects deposit/withdraw/borrow latency under load.

## Composability surface

- Receipt tokens are the main composability surface: whether a third-party protocol can accept them as collateral or trade them depends on the receipt token's redemption logic being decodable/stable — confirm datum schema and any transfer restrictions with source before building on it.
- Oracle dependency: borrow/liquidation logic depends on price feeds for every accepted collateral asset — confirm which oracle provider(s) back which assets, since a stale or unsupported feed blocks borrowing/liquidation for that asset (see `patterns/composability.md`).

## Gotchas

- Liquidation and collateral-factor parameters are asset-specific and change over time — do not hard-code values from any prior audit/doc without re-checking current parameters.
- Receipt token exchange rate is not 1:1 with the underlying and moves with accrued interest — don't assume face value equals redemption value.
