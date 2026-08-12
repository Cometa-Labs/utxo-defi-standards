# <Protocol Name>

`category:` <DEX/AMM | order-book DEX | money market | CDP/synthetics | liquid staking | other>
`last_verified:` YYYY-MM-DD
`source:` <links to docs/audits/source this entry is based on>
`maintained_by:` <agent/contributor, or "community — unverified">

## Mechanism

How it actually works, 3-6 lines. Not the pitch — the contract/datum-level mechanics.

## Deployments and tooling

Track where agents and integrators can find app frontends, test deployments, skills, MCP servers, SDK/API packages, and system parameter JSON. Use `Not found`, `Not available`, or `unknown` rather than guessing.

| Mainnet | Preprod | Preview | Skills | MCP | SDK/API | System param JSON |
|---|---|---|---|---|---|---|
| <mainnet app/API URL> | <preprod URL/status> | <preview URL/status> | <skills repo/package> | <MCP repo/package> | <SDK/API docs/package> | <config JSON URL(s)> |

## Token standards used

What it mints, and how (e.g. "LP tokens: plain native asset, no metadata standard" or "position receipts: CIP-68").

## On-chain identifiers

Use tables for every load-bearing policy, asset name, validator, state NFT, receipt token, or script address. Prefer AdaStat links for Cardano addresses and policies.

| Role | Policy / address | Asset name hex | Asset name text | Notes |
|---|---|---|---|---|
| <state NFT / LP token / receipt token / validator> | <AdaStat-linked policy or address> | <hex, or `none`> | <decoded text, or `unknown`> | <scope, version, source caveat> |

## Concurrency approach

Batcher / order-book / multiple pool UTXOs / Hydra / accepts contention / other. See `cardano/concurrency-patterns.md`.

## Composability surface

What an external protocol can hook into:
- Can its LP/receipt tokens be used as collateral elsewhere?
- Is its datum schema published and stable, or reverse-engineered/unstable?
- Is any state exposed via a reference-input-readable UTXO?

## Tx3 interface

Use `tools/tx3-interface.md`. Use `Not available` when the protocol has no known `.tx3` or TII artifact.

`status:` not_available | draft | usable | verified
`tx3_version:` <v1beta0 | unknown>
`last_generated:` YYYY-MM-DD | unknown

| Artifact | Path / URL | Version | Notes |
|---|---|---|---|
| Tx3 source | <path / URL / Not available> | <version / unknown> | <human-reviewed templates, source caveat, or Not available> |
| TII | <path / URL / Not available> | <version / unknown> | <generated artifact, digest, or Not available> |
| Tests | <path / URL / Not available> | <version / unknown> | <devnet/fixture coverage or Not available> |

### Templates

| Template | Standard action | Settlement model | Requires reference input | Position token impact | Notes |
|---|---|---|---|---|---|
| <tx name / Not available> | <Cardamom action vocabulary / unknown> | <atomic / batcher-order / batcher-settlement / unknown> | <yes / no / unknown> | <mints LP / burns receipt / none / unknown> | <schema caveats, version, source> |

## Gotchas

Things that bite integrators specifically — settlement delay windows, datum versioning breaks, fee quirks, minimum position sizes, etc.
