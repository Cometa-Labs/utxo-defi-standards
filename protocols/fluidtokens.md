# FluidTokens

`category:` money market (NFT-collateralised lending)
`last_verified:` 2026-07-21
`source:` reverse-engineered from on-chain transaction analysis (script hashes, config-datum-derived, verified against real mainnet txs) — not from FluidTokens' own docs/team. Treat mechanism claims as empirically observed behavior, not confirmed protocol design.
`maintained_by:` community — unverified, needs FluidTokens' own agent/team to confirm and correct

## Mechanism

NFT-collateralised P2P lending: lenders make ADA offers, borrowers post an NFT as collateral against an offer and receive ADA; repayment burns the loan position and returns the NFT, non-repayment past expiry lets the lender foreclose and claim the NFT. Two generations of contracts coexist on mainnet with materially different addressing:
- **V1/V2 (legacy):** enterprise script addresses (no staking credential) — `offer` and `loan` are each a single shared script address for the whole protocol. Still holds live positions; not just historical.
- **V3 (current):** follows a general-spend pattern (CIP-113-style) — every UTXO type (`loan`, `request`, `pool`, `bond`, `general_spend`, `config`) uses `Script(validatorHash)` as the payment credential with an **unconstrained staking credential** (pool/request UTXOs can carry any stake part; loan UTXOs carry the borrower's own stake key). There is no shared bech32 prefix across V3 UTXOs, so address-set matching (which works for V1/V2) does not work for V3 — detection has to be by redeemer/mint script-hash instead.

## Token standards used

- Loan/request position NFTs: minted/burned by the respective V3 validators as positions open/close — function as lifecycle markers, not user-facing receipts. No CIP-68 metadata observed.
- V3 config NFT: policy = `config` validator hash `64405c92b2641c1e83a62a553cfcba414725c35e5bed7b9ccb2ba7f5`, asset name is the literal hex-encoded string `parameters`. Its inline datum holds the *deployed* validator hashes for the other five V3 scripts.
- **Important:** V3 script hashes must be read from this on-chain config UTXO, not computed from a local `plutus.json` — the Aiken contracts are parameterised at deployment, so a locally-compiled hash will not match the mainnet-deployed one.

## Concurrency approach

No batcher — V3 is direct P2P: a `request` UTXO (borrower's ask) and a `pool`/offer-side UTXO are matched and spent directly, with `bond` and `general_spend` UTXOs handling the loan lifecycle in between. Each loan is its own UTXO once active, so concurrent unrelated loans don't contend with each other, but matching a specific request against a specific offer is still a single-UTXO-consumption race between whoever's tx lands first (see `cardano/concurrency-patterns.md`).

## Composability surface

- V3's `Script(validatorHash)` addressing with unconstrained staking credentials means the correct way to enumerate all UTXOs under a validator — regardless of what staking part they carry — is `GET /scripts/{scriptHash}/utxos`, not an address-based query. This is the only reliable way to find live positions for V3.
- The config UTXO's datum is the canonical source for current validator hashes — an external integrator should read it fresh rather than hard-coding V3 hashes, since a redeploy changes them.
- Loan/request UTXO inline datums appear to embed the counterparty's payment credential (confirmed usable via substring match on the hex datum for position lookups), but full ABI/datum schema has not been decoded — treat any datum-field-level integration as low-confidence until decoded further.

## Gotchas

- V1/V2's loan script address can return HTTP 400 from Blockfrost's UTXO endpoint (treat identically to 404/empty) rather than a normal empty list.
- V3's mint/burn events are the definitive classification signal (request-mint-only = borrow request opened, loan-mint = loan funded, loan-burn+NFT-to-user = repaid, loan-burn+no-NFT = foreclosed, request-burn-only = cancelled) — but Blockfrost's `/txs/{hash}/mints` returns HTTP 400 for these Plutus-script mints, so `tx.mint` comes back empty and classification must fall back to a net-ADA-flow heuristic instead.
- In that ADA-flow fallback, a *negative* net flow to the script (ADA returned to the user) can only mean a cancellation/withdrawal — a borrower never receives ADA directly back from the script in a way that looks like this; loan proceeds arrive in a separate, distinct tx. Don't classify a negative-flow tx as `borrow`.
- V1 and V2 share the same enterprise addresses in this analysis but are versioned separately in FluidTokens' own system — confirm whether V1 and V2 are actually address-identical or just observed as such in the sampled data before assuming permanently.
