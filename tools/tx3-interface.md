# Tx3 Interface Artifacts

`category:` protocol interface / transaction-template standard
`last_verified:` 2026-08-11
`source:` Tx3 GitHub `https://github.com/tx3-lang/tx3`
`maintained_by:` community — unverified, needs Tx3 maintainers/protocol agents to confirm current toolchain details

## What Tx3 is

Tx3 is an interface description language for UTxO protocols. It describes off-chain transaction construction: which UTxOs to spend or reference, which outputs to create, which datums/redeemers to attach, which assets to mint/burn, and which signers/collateral/validity constraints are needed.

Tx3 is not a smart contract language. Plutus/Aiken validators remain the on-chain source of truth.

## Why this belongs in protocol entries

UTxO DeFi lacks an ABI-like shared interface. A protocol's callable surface is a set of transaction shapes, not contract methods.

Use Tx3 artifacts to make those shapes reusable:

| Artifact | Role | Agent use |
|---|---|---|
| `.tx3` source | Human-readable canonical transaction templates | Review protocol action shape, datum/redeemer fields, policies, inputs/outputs |
| TII JSON | Generated machine-readable invocation interface | Generate clients, inspect parameter schemas, map parties, load protocol dynamically |
| TRP endpoint | Resolver/submission protocol | Resolve args + parties + chain state into unsigned transaction bytes |
| Devnet tests | Conformance evidence | Verify templates resolve/submit in controlled scenarios |

## Required protocol-entry slot

Every protocol entry MAY include this section. Use `Not available` when no Tx3 artifact exists.

```markdown
## Tx3 interface

`status:` not_available | draft | usable | verified
`tx3_version:` <v1beta0 | unknown>
`last_generated:` YYYY-MM-DD | unknown

| Artifact | Path / URL | Version | Notes |
|---|---|---|---|
| Tx3 source | `interfaces/<protocol>/<version>.tx3` | <version> | Human-reviewed templates |
| TII | `interfaces/<protocol>/<version>.tii.json` | <version> | Generated interface artifact |
| Tests | `interfaces/<protocol>/tests/` | <version> | Devnet or fixture conformance tests |

### Templates

| Template | Standard action | Settlement model | Requires reference input | Position token impact | Notes |
|---|---|---|---|---|---|
| `<tx_name>` | `swap` / `lend` / `borrow` / etc. | batcher/order / atomic / settlement / unknown | yes / no / unknown | mints LP / burns receipt / none / unknown | <caveats> |
```

## Standard action mapping

Prefer Cardamom's normalized action vocabulary when mapping Tx3 templates:

| Standard action | Use for |
|---|---|
| `swap` | Token exchange, including order submission and settlement rows |
| `add_liquidity` | Pool deposit / LP mint flows |
| `remove_liquidity` | Pool withdrawal / LP burn flows |
| `stake` | Staking/farm deposit of position tokens |
| `unstake` | Staking/farm withdrawal |
| `lend` | Supply/deposit into money market or lending pool |
| `borrow` | Borrow/open debt position |
| `repay` | Repay debt |
| `withdraw` | Redeem supplied asset or claim lent funds |
| `open_cdp` | Create collateralized debt position |
| `add_collateral` | Add collateral to an existing position |
| `mint` | Mint protocol asset/synthetic/receipt where no narrower action fits |
| `burn` | Burn protocol asset/synthetic/receipt where no narrower action fits |
| `liquidation` | Liquidation spend/settlement |
| `claim_reward` | Reward claim |
| `unknown_protocol_interaction` | Detectable protocol interaction with unknown action |

If a protocol has separate submit/settle phases, keep both templates but map them to the same standard action and distinguish them in `Settlement model` / `Notes`.

## Authoring guidance for protocol agents

When adding a Tx3 interface:

- Include one template per externally meaningful action.
- Split batcher/order-submit and batcher-settlement flows.
- Declare load-bearing policies, script hashes, reference-script UTxOs, LP/receipt-token policies, and state NFTs.
- Type datums and redeemers where the schema is public or source-backed.
- Mark reverse-engineered datums/redeemers as `draft` or `partial`, not `verified`.
- Use doc comments in `.tx3`; downstream TII consumers can surface them in generated clients and agent tools.
- Include resolver assumptions: required reference inputs, collateral, signer parties, validity windows, oracle inputs, and API dependencies.
- Include tests or fixture hashes for each template when available.

## Consumer guidance for app/agent developers

Use Tx3 artifacts in this order:

1. Read the protocol entry for mechanism, identifiers, concurrency model, and gotchas.
2. Inspect `.tx3` source for transaction shape and datum/redeemer types.
3. Load TII for parameter schemas, parties, and generated clients.
4. Use TRP only when resolving/building transactions; do not infer historical activity solely from TII.
5. For indexing, match historical transactions against template evidence progressively: policy/script match, input/output role match, mint/burn match, datum/redeemer match, value-flow match.
6. Preserve confidence labels when evidence is incomplete.

## Verification levels

| Status | Meaning |
|---|---|
| `not_available` | No Tx3 artifact is known for this protocol |
| `draft` | Template exists but is incomplete, untested, or reverse-engineered |
| `usable` | Source-backed enough for read-path matching and cautious app integration |
| `verified` | Confirmed by protocol maintainers or tested against source/docs and devnet/mainnet fixtures |

## Version-alignment caveat

Tx3 workflow docs and tool support may move faster than the v1beta0 spec artifacts. Before depending on a feature, verify the current Tx3 toolchain supports it.

Check especially:

- `trix` command availability and flags;
- TII schema version;
- SDK/codegen support by language;
- TRP endpoint behavior;
- arithmetic operators beyond `+` / `-`;
- dynamic asset construction;
- input filters/ranking;
- conditionals and iteration;
- devnet/Dolos test syntax.
