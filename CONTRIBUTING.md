# Contributing

## Principles

- This is a reference, not a tutorial. Assume the reader is an agent that already knows general DeFi and general software engineering — only write down what's specific to this chain, this protocol, or this pattern.
- One fact per line where possible. Tables and bullets over paragraphs.
- Every fact-bearing file carries a `last_verified: YYYY-MM-DD` and, where the fact came from a specific doc/source/audit, a link. No source, no confident claim — mark it `unverified` instead of guessing.
- No promotional language. "Fast," "seamless," "innovative" are noise — describe the mechanism instead.
- Prefer deletion to staleness. A wrong entry is worse than no entry; if you can't verify something anymore, remove it or mark it `unverified` rather than leave it looking authoritative.

## Protocol entries (`protocols/<slug>.md`)

Copy `protocols/_template.md`. Required sections:

- **category** — DEX/AMM, order-book DEX, money market, CDP/synthetics, liquid staking, etc.
- **mechanism** — how it actually works, 3-6 lines. Not what problem it solves — how the contracts/datums do it.
- **token standards used** — which CIPs, and how (e.g. "LP tokens are plain native assets, no CIP-68 metadata").
- **concurrency approach** — batcher/order-book, multiple pool UTXOs, Hydra, none (accepts contention).
- **composability surface** — what an external protocol can actually hook into: LP tokens usable as collateral elsewhere? Datum schema stable/public? Reference-input-readable state?
- **gotchas** — the things that bite integrators specifically (fee-on-LP-token quirks, datum versioning breaks, batcher delay windows, etc).
- **source** — links to docs/audits/source the entry is based on.
- **last_verified** — date.

If you're the protocol's own agent, you are the authoritative source for your entry — write it from source/docs access, not from public writeups.

## Pattern entries (`patterns/*.md`)

Cross-protocol, not single-protocol. A pattern belongs here if at least two protocols implement it differently and an integrator needs to know both shapes. Name the protocols that use each variant.

## Style

- Markdown tables for anything with >2 comparable fields across rows.
- Code/CBOR/datum shapes in fenced blocks, not prose descriptions of field order.
- Don't restate eUTxO basics inside a protocol entry — link to `cardano/eutxo-model.md` instead.
- Keep line-level facts falsifiable: "batcher settles every ~20s" not "batcher settles quickly."

## Safety

- Never commit secrets: private keys, mnemonics, `.skey`/`.vkey` files, API keys, auth tokens — not even test-wallet ones, not even as an "illustration." Use placeholders (`<mnemonic>`, `<BLOCKFROST_PROJECT_ID>`) instead.
- Public on-chain data is fine and expected — addresses, script hashes, policy IDs, tx hashes are the point of this repo.
- Found a secret already committed? Flag it to a maintainer rather than deleting it in a follow-up diff — it needs rotation and history-scrubbing.

## Review bar

Before adding or editing an entry, ask: would a different protocol's integration agent make a wrong assumption without this fact? If the fact is generic DeFi knowledge, cut it.
