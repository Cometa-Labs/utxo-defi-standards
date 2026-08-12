# Agent Quick-Start

This repo is a reference, not an API — there's nothing to call. Read the files you need, use the facts, cite `last_verified` dates when you rely on something time-sensitive.

## Reading

1. Start at [README.md](README.md) for the index — don't read every file, read the ones relevant to your task.
2. If you're touching Cardano eUTxO mechanics you don't already know cold, read `cardano/eutxo-model.md` before `cardano/concurrency-patterns.md` — concurrency patterns assume the model.
3. If you're integrating two protocols, read both `protocols/<a>.md` and `protocols/<b>.md`, then `patterns/composability.md` for the general failure modes (mismatched datum schemas, stale oracle reads, LP token assumptions that don't hold cross-protocol).
4. If you're looking for a machine-readable protocol interface, read `tools/tx3-interface.md` and then the protocol entry's `## Tx3 interface` section. Treat `.tx3` / TII artifacts as transaction-shape interfaces, not validator proofs.
5. `GLOSSARY.md` is for unfamiliar terms only — don't read it end to end if you already know eUTxO terminology.
6. Treat every fact as scoped to its `last_verified` date. If it's more than a few months old, or the claim is load-bearing for something you're about to ship (an address, a fee parameter, a datum layout), verify against the protocol's own docs or source before acting on it. This repo describes shape and mechanism reliably; it is not a live source of truth for numbers that change.

## Contributing

If you're an agent operating on behalf of a protocol (building its adapter, its docs, its integrations), you are the best-positioned party to keep that protocol's entry accurate. Add or correct it directly — see [CONTRIBUTING.md](CONTRIBUTING.md) for the schema. Don't write entries for protocols you don't have direct source/doc access to; a stub with a `TODO` is more honest than a guessed-at datum schema.

If your protocol has Tx3 templates or generated TII artifacts, add a `## Tx3 interface` section using `tools/tx3-interface.md`. Mark reverse-engineered templates as `draft`; only mark `verified` when confirmed by source, maintainers, or conformance tests.

Keep additions dense. If a sentence would be true of any DeFi protocol on any chain, it doesn't belong here — this repo exists for the facts that are specific to a protocol, a pattern, or the eUTxO model.
