# FluidTokens

`category:` money market (NFT-collateralised lending)
`last_verified:` 2026-07-22
`source:` FluidTokens docs `https://docs.fluidtokens.com/`; Cardano lending docs `https://docs.fluidtokens.com/cardano/lending/`; developer docs `https://docs.fluidtokens.com/developers/`; AdaStat address/policy pages linked below; reverse-engineered on-chain analysis for exact V1/V2/V3 identifiers not yet found in official docs
`maintained_by:` community — unverified, needs FluidTokens' own agent/team to confirm and correct

## Mechanism

NFT-collateralised P2P lending: lenders make ADA offers, borrowers post an NFT as collateral against an offer and receive ADA; repayment burns the loan position and returns the NFT, non-repayment past expiry lets the lender foreclose and claim the NFT. Two generations of contracts coexist on mainnet with materially different addressing:
- **V1/V2 (legacy):** enterprise script addresses (no staking credential) — `offer` and `loan` are each a single shared script address for the whole protocol. Still holds live positions; not just historical.
- **V3 (current):** follows a general-spend pattern (CIP-113-style) — every UTXO type (`loan`, `request`, `pool`, `bond`, `general_spend`, `config`) uses `Script(validatorHash)` as the payment credential with an **unconstrained staking credential** (pool/request UTXOs can carry any stake part; loan UTXOs carry the borrower's own stake key). There is no shared bech32 prefix across V3 UTXOs, so address-set matching (which works for V1/V2) does not work for V3 — detection has to be by redeemer/mint script-hash instead.

## Deployments and tooling

| Mainnet | Preprod | Preview | Skills | MCP | SDK/API | System param JSON |
|---|---|---|---|---|---|---|
| Not recorded in this entry | No data | [`dev.fluidtokens.com/ada/markets`](https://dev.fluidtokens.com/ada/markets) | Not found | Not found | [`docs.fluidtokens.com/developers`](https://docs.fluidtokens.com/developers/) — no public Cardano SDK package identified | Not found |

## Token standards used

- Loan/request position NFTs: minted/burned by the respective V3 validators as positions open/close — function as lifecycle markers, not user-facing receipts. No CIP-68 metadata observed.
- V3 deployed config NFT observed on-chain: policy [`219832152b2c489358f4c02a1818d312a851b1f55774ae881e33a907`](https://adastat.net/policies/219832152b2c489358f4c02a1818d312a851b1f55774ae881e33a907), asset name `parameters`, at [`addr1wysesvs49vky3y6c7nqz5xqc6vf2s5d374thft5grce6jpcwela6v`](https://adastat.net/addresses/addr1wysesvs49vky3y6c7nqz5xqc6vf2s5d374thft5grce6jpcwela6v). Its inline datum holds deployed validator hashes; do not substitute unparameterised `plutus.json` hashes.
- Previously observed/unparameterised config validator hash [`64405c92b2641c1e83a62a553cfcba414725c35e5bed7b9ccb2ba7f5`](https://adastat.net/policies/64405c92b2641c1e83a62a553cfcba414725c35e5bed7b9ccb2ba7f5) appears in older investigation notes; verify against the live config NFT before use.
- **Important:** V3 script hashes must be read from this on-chain config UTXO, not computed from a local `plutus.json` — the Aiken contracts are parameterised at deployment, so a locally-compiled hash will not match the mainnet-deployed one.

## On-chain identifiers

| Role | Policy / address | Asset name hex | Asset name text | Notes |
|---|---|---|---|---|
| V1/V2 offer address | [`addr1w8dl44wfq4e8s8aaz9t8g3l2nhg4jxhm0qr2nppfxewlz3qkstxnw`](https://adastat.net/addresses/addr1w8dl44wfq4e8s8aaz9t8g3l2nhg4jxhm0qr2nppfxewlz3qkstxnw) | `none` | `none` | Legacy shared offer script address. |
| V1/V2 loan address | [`addr1wxjfv7fgjnq53lnde29dyncmyzwtt5v6wqnzlcrk8l5vxuqkh0kz3`](https://adastat.net/addresses/addr1wxjfv7fgjnq53lnde29dyncmyzwtt5v6wqnzlcrk8l5vxuqkh0kz3) | `none` | `none` | Legacy shared loan script address. |
| V3 config NFT | [`219832152b2c489358f4c02a1818d312a851b1f55774ae881e33a907`](https://adastat.net/policies/219832152b2c489358f4c02a1818d312a851b1f55774ae881e33a907) | `706172616d6574657273` | `parameters` | Canonical deployed config UTxO at [`addr1wysesvs49vky3y6c7nqz5xqc6vf2s5d374thft5grce6jpcwela6v`](https://adastat.net/addresses/addr1wysesvs49vky3y6c7nqz5xqc6vf2s5d374thft5grce6jpcwela6v). |
| V3 pool script hash | `fca77bcce1e5e73c97a0bfa8c90f7cd2faff6fd6ed5b6fec1c04eefa` | `none` | `none` | Config datum index 0 in observed deployment. |
| V3 pool stake hash | `d5940dc2ad9e3f43544fb02a28240f215602603a7519588532b72c53` | `none` | `none` | Config datum index 1 in observed deployment. |
| V3 request script hash | `befbcb19919ff8ce5323d123c835da8e7653a098ad482271a72b72f2` | request NFT name is position-specific | position-specific | Config datum index 2 in observed deployment. |
| V3 loan script hash | `a37578f027ae878115cc70cd0909ddc855d67b6dd3bd038a757bd221` | loan NFT name is position-specific | position-specific | Config datum index 3 in observed deployment. |
| V3 bond script hash | `eadc69a5d2d1357acc9b9d49ec5390fcdf6e080c7a40139917223dcb` | bond NFT name is position-specific | position-specific | Config datum index 4 in observed deployment. |

Use the config datum as the canonical source rather than copying a static script-hash list blindly.

## Concurrency approach

No batcher — V3 is direct P2P: a `request` UTXO (borrower's ask) and a `pool`/offer-side UTXO are matched and spent directly, with `bond` and `general_spend` UTXOs handling the loan lifecycle in between. Each loan is its own UTXO once active, so concurrent unrelated loans don't contend with each other, but matching a specific request against a specific offer is still a single-UTXO-consumption race between whoever's tx lands first (see `cardano/concurrency-patterns.md`).

## Composability surface

- V3's `Script(validatorHash)` addressing with unconstrained staking credentials means the correct way to enumerate all UTXOs under a validator — regardless of what staking part they carry — is `GET /scripts/{scriptHash}/utxos`, not an address-based query. This is the only reliable way to find live positions for V3.
- The config UTXO's datum is the canonical source for current validator hashes — an external integrator should read it fresh rather than hard-coding V3 hashes, since a redeploy changes them.
- Loan/request UTXO inline datums appear to embed the counterparty's payment credential (confirmed usable via substring match on the hex datum for position lookups), but full ABI/datum schema has not been decoded — treat any datum-field-level integration as low-confidence until decoded further.
- Partial request datum structure observed from V3 request UTxOs:

```text
RequestDatum = Constr(0)[
  ...,
  loan_terms: Constr(0)[..., ..., interest_bps, ...],
  ...,
  accepted_collaterals: [Constr(_)[policy_id, asset_name_constr, ...]],
  ltvs: [uint],
  ...
]
```

This shape is partial: it only identifies `interestBps`, collateral policy/name pairs, and LTV entries from observed V3 request datums. It is not a full schema.

## SDK/API

- Official docs: `https://docs.fluidtokens.com/`; Cardano lending docs: `https://docs.fluidtokens.com/cardano/lending/`.
- Developer hub: `https://docs.fluidtokens.com/developers/`.
- No public Cardano SDK package was identified in the official docs during verification. For write-path integrations, use the developer docs and confirm current contract schemas with FluidTokens directly.

## Gotchas

- V1/V2's loan script address can return HTTP 400 from Blockfrost's UTXO endpoint (treat identically to 404/empty) rather than a normal empty list.
- V3's mint/burn events are the definitive classification signal (request-mint-only = borrow request opened, loan-mint = loan funded, loan-burn+NFT-to-user = repaid, loan-burn+no-NFT = foreclosed, request-burn-only = cancelled) — but Blockfrost's `/txs/{hash}/mints` returns HTTP 400 for these Plutus-script mints, so `tx.mint` comes back empty and classification must fall back to a net-ADA-flow heuristic instead.
- In that ADA-flow fallback, a *negative* net flow to the script (ADA returned to the user) can only mean a cancellation/withdrawal — a borrower never receives ADA directly back from the script in a way that looks like this; loan proceeds arrive in a separate, distinct tx. Don't classify a negative-flow tx as `borrow`.
- V1 and V2 share the same enterprise addresses in this analysis but are versioned separately in FluidTokens' own system — confirm whether V1 and V2 are actually address-identical or just observed as such in the sampled data before assuming permanently.
