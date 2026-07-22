# Other UTXO Chains

`last_verified: 2026-07-21`

Bitcoin and Ergo, for contrast and for wherever interop actually crosses chains. Not maintained to the same depth as the Cardano-focused files — treat as directional, verify specifics before relying on them.

## Bitcoin

Bitcoin base-layer UTXOs carry no datum and are locked only by Script (no general-purpose validators) — there's no native equivalent of eUTxO's arbitrary-state-plus-validator model. "DeFi" on Bitcoin is built either off-chain or via metaprotocols layered on top of plain UTXOs:

- **Lightning Network** — bidirectional payment channels for fast/cheap BTC transfer. Payments, not general DeFi (no lending/AMM logic natively).
- **Discreet Log Contracts (DLCs)** — oracle-signed outcome contracts between two parties, settled via Bitcoin script + adaptor signatures. Enables derivatives/prediction-market-style payoffs without putting contract logic on-chain; the "contract" is really a pre-agreed set of possible on-chain outcomes signed by an oracle.
- **RGB** — client-side-validated smart contracts: contract state and logic live off-chain (held by participants), with Bitcoin UTXOs used only as a commitment/anchoring layer (via single-use-seals). Conceptually closer to eUTxO's "UTXO as state pointer" idea than plain Bitcoin script, but validation is done by clients, not miners/nodes.
- **Taproot Assets** (formerly Taro) — asset issuance anchored in Taproot outputs, designed to interoperate with Lightning for asset transfer, not just BTC.
- **Runes / BRC-20** — fungible-token metaprotocols built entirely on transaction/inscription conventions (no on-chain enforcement beyond what indexers agree to interpret). No validator logic at all — a token's rules are a social/indexer convention, not consensus-enforced. Materially weaker guarantees than a Cardano native asset's policy-script-enforced minting.

Practical note: none of these give Bitcoin a direct eUTxO-style "datum + validator" composability model. Any "Bitcoin DeFi" claim should be checked against which of the above it actually uses, since the trust/enforcement model differs sharply between them (consensus-enforced vs. indexer-convention vs. client-side-validated vs. oracle-signed).

## Ergo

- Also an eUTxO chain (same lineage of ideas as Cardano — Ergo predates Cardano's eUTxO formalization and the "eUTxO" academic model was written up drawing on both). Contracts written in ErgoScript, compiled to Sigma protocols (a form of sigma-protocol-based script, not EVM-style bytecode).
- Key design difference from Cardano: Ergo boxes (Ergo's term for UTXOs) support **self-replicating** contract patterns more directly in common practice — a box's script can require that spending it produces a new box with a similar script, making long-lived stateful contracts (like an AMM pool) a more idiomatic single-box pattern. Cardano can express the same idea (a validator requiring its own address/datum-shape to persist across the spend) but it's a convention enforced by the validator author, not a distinct language feature.
- DEX activity on Ergo (e.g. Spectrum, cross-chain-oriented) has historically also had to solve the same shared-pool-contention problem as Cardano AMMs — check a specific protocol's own docs for its concurrency approach rather than assuming it matches a Cardano pattern.

## Cross-UTXO-chain interop

- No native cross-chain messaging between separate UTXO chains (or between a UTXO chain and an account-model chain) — every bridge is an added trust assumption on top of both chains' base security.
- Common bridge trust models: federated/multisig custody (a permissioned set of signers attests to lock/mint events), light-client/SPV-proof based (more trust-minimized but heavier to implement and not available for every chain pair), or a dedicated sidechain with its own validator set (e.g. EVM-compatible sidechains anchored to Cardano) that takes on separate security assumptions from Cardano mainnet itself.
- When evaluating a "cross-chain" DeFi claim, identify which bridge/trust model backs the wrapped asset in question — a wrapped asset's risk profile is a function of its bridge, not of the origin chain's security.
