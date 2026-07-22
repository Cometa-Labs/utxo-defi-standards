# Glossary

UTXO/eUTxO terms as used elsewhere in this repo. Not exhaustive — only terms that show up in composability discussions.

| Term | Meaning |
|---|---|
| UTXO | Unspent Transaction Output. A discrete chunk of value that can only be consumed (spent) whole, as a single input, by exactly one transaction. |
| eUTxO | Extended UTXO (Cardano). A UTXO can carry a **datum** (arbitrary state) and be locked by a **validator script**, not just a public key. |
| Datum | Data attached to a UTXO. Either inlined in the UTXO (post-Vasil) or referenced by hash (pre-Vasil / still common), resolved by including the matching data in the tx. |
| Redeemer | Data supplied by the *spending* transaction, passed to the validator alongside the datum, to justify/parameterize the spend (e.g. "swap" vs "cancel"). |
| Validator | A script that returns pass/fail given (datum, redeemer, script context). Does not "run" a state transition — it only approves or rejects the transaction that spends the UTXO it locks. |
| Native token / native asset | A token that lives directly in the ledger's multi-asset value type, not a separate contract. Identified by `policyId + assetName`, no ERC-20-style deployed contract per token. |
| Policy ID | Hash of a token's minting policy script. Fixed forever per policy; the same policy can mint many asset names. |
| Asset fingerprint | CIP-14 bech32 hash of `policyId + assetName`, used as a stable dedup key across UIs/indexers. |
| Min-ADA / min-UTXO | Every UTXO must hold a minimum ADA amount (scales with how much data/tokens it carries). Protocols must budget for this on every UTXO they create. |
| Collateral input | A UTXO set aside to pay fees if a script validation fails at phase-2. Required for any tx that runs a Plutus script. |
| Reference input | A UTXO a transaction can *read* without spending (post-Vasil, CIP-31). Lets many transactions read shared state (e.g. an oracle price) with zero contention. |
| Reference script | A script attached to a UTXO for other transactions to reference by input instead of re-including the full script bytes (CIP-33) — cuts tx size/fees. |
| Contention / UTXO collision | Two transactions in the same block both trying to spend the same UTXO — only one succeeds, the other fails. The core reason naive shared-pool AMMs don't scale on eUTxO without a workaround. |
| Batcher | An off-chain (or semi-trusted on-chain-triggered) process that collects many user "order" UTXOs and settles them against a shared pool in one transaction, avoiding per-user contention on the pool UTXO. |
| Order UTXO | A UTXO a user creates expressing intent (swap, deposit) that a batcher later consumes and fulfils, rather than the user interacting with the pool UTXO directly. |
| LP token | Native asset minted to represent a share of a liquidity pool. Transferable, and — if the issuing protocol allows it — usable as collateral or an input elsewhere. |
| CDP | Collateralized Debt Position — lock collateral, mint a synthetic/stable asset against it (Indigo, Djed-style protocols). |
| Hydra | Cardano's isomorphic state-channel L2. A "head" runs fast/cheap off-chain transactions among fixed participants, settling to L1 on open/close. |
| DRep | Delegated Representative — a Cardano governance (CIP-1694) role, unrelated to DeFi mechanics but sometimes conflated with staking pools; not covered further here. |
