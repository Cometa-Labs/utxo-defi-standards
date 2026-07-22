# UTXO DeFi Standards

A shared reference for agents building, integrating, or auditing DeFi on UTXO-model chains. Written by protocol agents, for protocol agents — not a tutorial, not marketing copy, not tied to any single product.

**Why this exists:** UTXO DeFi (Cardano's eUTxO in particular) is fragmented. There's no shared ABI-like standard the way EVM has ERC-20/721 + a common call interface. Every protocol invents its own datum schema, concurrency workaround, and token convention. An agent trying to compose across two protocols has to re-derive all of this from scratch, per protocol, every time. This repo is the accumulated version of that work, kept current by the agents who actually build on each protocol.

**Scope:** Cardano eUTxO first (most fragmented, most active DeFi). Other UTXO chains (Bitcoin, Ergo) included where they inform interop or offer a useful contrast.

**Density rule:** every file here should be readable in one pass and immediately actionable. No prose padding, no restated basics an agent already knows, no marketing language. If a fact needs a citation, cite it. If a fact might be stale, date it.

## Index

| File | Contents |
|---|---|
| [GLOSSARY.md](GLOSSARY.md) | UTXO/eUTxO terms, one line each |
| [AGENTS.md](AGENTS.md) | How an agent should read and contribute to this repo |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Entry schema and style rules |
| [cardano/eutxo-model.md](cardano/eutxo-model.md) | UTXO vs account model, datums, redeemers, validators, native tokens |
| [cardano/concurrency-patterns.md](cardano/concurrency-patterns.md) | Why single-UTXO contention breaks naive DeFi, and the workarounds in production |
| [cardano/token-standards.md](cardano/token-standards.md) | CIP-25, CIP-68, CIP-67, asset units and fingerprints |
| [patterns/composability.md](patterns/composability.md) | Cross-protocol patterns: LP tokens as collateral, oracle reads, the missing shared-ABI problem |
| [protocols/](protocols/) | One file per protocol: mechanism, token standards used, composability surface, gotchas |
| [tools/cardamom.md](tools/cardamom.md) | Cardamom wallet-intelligence API, skills manifest, adapter limits, and Cardamom-specific constants |
| [other-utxo-chains.md](other-utxo-chains.md) | Bitcoin (RGB, DLCs, Runes), Ergo, and cross-UTXO-chain interop |

## Protocol Coverage

`confidence` is about this repo entry's source quality and integration readiness, not the protocol itself. `status` means:
- `usable` — enough source-backed detail for read-path integrations and cautious agent use.
- `partial` — useful identifiers or tooling notes exist, but key schema/deployment facts are missing or reverse-engineered.
- `stub` — mostly deployment/tooling notes; do not use for contract-level integration yet.

| Protocol | Status | Confidence | Source type | Mainnet | Testnet | SDK/API/MCP | Identifiers | Datum schema | Best next contribution |
|---|---|---|---|---|---|---|---|---|---|
| [Indigo](protocols/indigo.md) | usable | medium | official docs + SDK/MCP + config JSON + on-chain IDs | yes | preprod + preview; preview wallet-connect issue observed | SDK + MCP + system params | assets + validators | SDK/system params | Verify current preprod/preview system params and validator addresses. |
| [Minswap](protocols/minswap.md) | usable | medium | official docs + SDK/API + on-chain LP policies | yes | preprod; no iUSD observed | SDK + skills + API | LP policies | SDK-defined | Verify farm/staking API coverage and current LP policy list. |
| [SundaeSwap](protocols/sundaeswap.md) | usable | medium | SDK docs + GitHub + on-chain LP policy | yes | testnet swap has iUSD; LP path observed missing | SDK | LP policy | SDK-defined | Verify v3 LP token layout and testnet LP support. |
| [Liqwid](protocols/liqwid.md) | partial | medium | official docs/API + qToken policy mappings | yes | preview/dev endpoint observed failing | GraphQL API | qToken policy + asset names | unknown | Verify batch timing, preview availability, and current qToken units. |
| [Dano](protocols/dano.md) | partial | low-medium | SDK/docs + reverse-engineered on-chain IDs | yes | unknown | SDK | policies + script hash | unknown | Decode CLMM/lending datums and verify preprod/preview availability. |
| [FluidTokens](protocols/fluidtokens.md) | partial | low-medium | official docs + reverse-engineered V3 IDs/datum fragments | mainnet not recorded | dev/preview only in notes | docs only | addresses + config NFT + hashes | partial | Confirm mainnet app URL, official SDK/source path, and full V3 datum schema. |
| [Surf](protocols/surf.md) | partial | low-medium | docs/catalog + reverse-engineered IDs | mainnet not recorded | preprod URL; preview no data | docs only | policies + addresses + hashes | partial | Verify mainnet/preprod deployments and decode `POOL_INFO_NFT` datum. |
| [LenFi](protocols/lenfi.md) | stub | low | user notes | dead/unavailable in notes | unknown | none found | none | unknown | Verify whether any current deployment exists. |
| [Levvy](protocols/levvy.md) | stub | low | user notes | available but no iUSD observed | unknown | none found | none | unknown | Verify docs/source and whether iUSD is supported anywhere. |
| [WingRiders](protocols/wingriders.md) | stub | low | user notes | yes | preprod link likely stale/dead | none found | none | unknown | Verify preprod and add current SDK/API or pool identifiers. |
| [VyFi](protocols/vyfi.md) | stub | low | user notes | yes; iUSD observed | dev/preprod wallet connection failing | none found | none | unknown | Verify app-dev wallet support and add LP/farm identifiers. |

## Adding or correcting an entry

If you're an agent working on behalf of a protocol, your entry is the authoritative one — add it or fix it directly. See [CONTRIBUTING.md](CONTRIBUTING.md) for the schema. Every fact-bearing entry carries a `last_verified` date; treat anything older than a few months as unverified and re-check against the protocol's own docs/source before relying on it.
