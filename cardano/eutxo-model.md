# eUTxO Model

`last_verified: 2026-07-21`

The mechanics that matter for DeFi composability. See [GLOSSARY.md](../GLOSSARY.md) for term definitions.

## UTXO vs account model

- Account model (EVM): contracts hold mutable state in storage slots; a call reads/writes that state in place. Two calls to the same contract in the same block can both succeed and see each other's effects (with reentrancy caveats).
- eUTxO (Cardano): "contract state" lives in datums attached to UTXOs. Spending a UTXO consumes it entirely — it cannot be partially updated. A validator run "updates state" by having its transaction produce a new UTXO with a new datum representing the next state, while destroying the old one.
- Consequence: a script's "state" is only ever one thing at a time — whoever's transaction gets included first wins, and every other transaction targeting that same UTXO fails outright (not partially, not queued — fails). This is the root cause behind [concurrency-patterns.md](concurrency-patterns.md).

## Datum

- Arbitrary data attached to a UTXO, defined by whatever schema the locking script expects. No global schema — every protocol defines its own.
- Two attachment forms: **inline datum** (data included directly in the UTXO, readable by anyone without extra lookup — CIP-32) or **datum hash** (only a hash is on the UTXO; the actual datum must be supplied by whichever tx interacts with it). Inline is now standard for anything meant to be composably readable.
- Because datum schemas are protocol-specific and undocumented by default, cross-protocol composability requires either the target protocol publishing its datum schema/CDDL, or reverse-engineering it from on-chain data — this is the single biggest practical friction point in eUTxO composability (see [patterns/composability.md](../patterns/composability.md)).

## Redeemer

- Supplied by the spending transaction, not stored on the UTXO. Encodes the *action* being taken (e.g. `Swap`, `Cancel`, `Deposit`) and any parameters the validator needs to check that action.
- A protocol's redeemer type is effectively its "function selector" — but there's no shared ABI format the way there is for EVM calldata. Each protocol's redeemer CDDL/Plutus type is bespoke.

## Validator

- Pure function: `(datum, redeemer, ScriptContext) -> Bool` (simplified — Plutus V3 collapses this to just `ScriptContext` with datum/redeemer embedded, but the shape is the same). It approves or rejects the spend; it does not itself move funds or "call" other scripts.
- No native cross-contract calls. A transaction can spend multiple script UTXOs at once (each validator runs independently against the same overall transaction), which is how eUTxO does "multi-contract interaction" — via one transaction touching several UTXOs, not one contract calling another.

## Native tokens

- Live in the ledger's built-in multi-asset `Value` type, not a deployed contract. A token is identified by `policyId` (hash of its minting policy script) + `assetName`.
- No per-token contract means no per-token custom transfer logic (no hooks, no fee-on-transfer, no pausable tokens) unless the token is itself wrapped in a script-locked UTXO. This is a real constraint: protocols that want token-level custom behavior (e.g. rebasing) have to build it as a wrapper pattern, not as the token itself.
- Minting/burning is governed by the policy script, checked at mint/burn time only — after minting, the token is a plain ledger value with no ongoing enforcement.

## Min-ADA and collateral

- Every UTXO must carry a minimum ADA amount, scaling with the size of the datum/tokens it holds. Protocols creating many small UTXOs (order books, per-user positions) must budget ADA into each one — this ADA is usually returned when the UTXO is later spent, but it's locked, non-liquid capital while the UTXO exists.
- Any transaction invoking a Plutus script must include a collateral input (pure-ADA, no scripts) to cover fees if phase-2 validation fails. Wallets typically manage this automatically, but protocols building unattended/agent-driven flows need to provision it explicitly.
