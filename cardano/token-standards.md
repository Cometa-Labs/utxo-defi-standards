# Token Standards

`last_verified: 2026-07-21`

Cardano has no ERC-20/ERC-721 equivalent baked into a common interface — native assets are all the same underlying ledger primitive (see [eutxo-model.md](eutxo-model.md)). "Standards" here are metadata/labeling conventions layered on top, defined as CIPs (Cardano Improvement Proposals).

## Asset unit format

- A native asset's on-ledger identity is `policyId (28 bytes / 56 hex chars) + assetName (0-32 bytes, arbitrary)`. Concatenated hex, this is commonly called the asset's **unit**.
- **Asset fingerprint** (CIP-14): a bech32-encoded hash of `policyId + assetName`, used as a stable, human-comparable ID across tools/indexers/UIs when the raw hex unit is unwieldy or when you want a consistent identifier independent of how the asset name is encoded/displayed.

## CIP-25 — NFT metadata (off-chain)

- Convention for attaching NFT metadata via transaction metadata (label `721`) at mint time: JSON describing name, image, attributes, keyed by policy ID and asset name.
- Off-chain in the sense that it lives in transaction metadata, not in a datum — not directly readable by a validator script. Fine for marketplaces/wallets rendering an NFT; not usable if a DeFi contract needs to read/verify metadata on-chain.
- Still widely used for simple NFT collections where no protocol needs to reason about metadata in a script.

## CIP-68 — datum-based structured metadata

- Splits an asset into two tokens minted together: a **reference NFT** (holds the metadata as an inline datum on a UTXO controlled by the minting policy) and a **user token** (the actual fungible/NFT/RFT asset held by the owner).
- Metadata is on-chain, in a datum — scripts can read and even update it (by having the minting/update policy allow spending and recreating the reference NFT's UTXO with new metadata), unlike CIP-25's static off-chain blob.
- Asset name prefixes distinguish the token kind — reference NFT vs. fungible/NFT/RFT user token — via the numeric label scheme defined in CIP-67 (see below). Check a specific protocol's entry for which label it uses.
- This is the standard receipt/position-token pattern in newer Cardano DeFi: a protocol issuing an LP token or a lending receipt token with metadata that needs to be verifiable/updatable on-chain will typically use CIP-68 rather than a bare unlabeled native asset.

## CIP-67 — asset name label registry

- Defines the convention for a numeric label (e.g. `100`, `222`, `333`, `444`) prefixed onto an asset name to signal its type/purpose (reference NFT, fungible token, NFT, rich-FT) — this is what CIP-68 relies on to distinguish its two paired tokens, and is reused by other proposals that need a labeled-asset-name convention.
- If you see an asset name starting with a bracketed decimal-looking prefix in its bech32/hex encoding, check CIP-67 for what that label means before assuming it's arbitrary.

## Practical note for integrators

- A native asset carries none of this by default — CIP-25/CIP-68 compliance is opt-in per minting policy. Don't assume any given token follows either standard; check the specific protocol's entry under `protocols/` for what it actually mints, and treat "has no metadata standard, just a bare unit" as a normal, common case, not an error state.
