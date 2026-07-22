# Cardamom

`category:` wallet intelligence / protocol-indexing tool
`last_verified:` 2026-07-22
`source:` local repo `../cardamom`; public skills manifest `https://cardamom.live/api/skills`; integration standard `https://cardamom.live/protocol-standard`
`maintained_by:` Cardamom app team / local source

## What it is

Cardamom is not an on-chain protocol. It is a Next.js/TypeScript address-intelligence app that indexes Cardano wallet history, recognises supported DeFi protocol interactions, and exposes normalized activity/position data for agents and web clients.

## Agent surface

- Skills manifest: `GET https://cardamom.live/api/skills`.
- Per-protocol endpoint: `GET https://cardamom.live/api/skills/{protocol}?address={addr}&page=1&pageSize=20`.
- Supported protocol slugs: `sundaeswap`, `minswap`, `fluidtoken`, `indigo`, `liqwid`, `dano`, `surflend`.
- Public endpoints are unauthenticated and rate-limited; `X-Api-Key` can be used for higher limits.
- Cardamom does not ship an MCP server in the local repo. Use the HTTP skills manifest directly, or wrap those endpoints in an external MCP server.

## Adapter contract

Cardamom adapters implement:

```ts
type ProtocolAdapter = {
  protocol: string;
  detect(tx: CardanoTransaction, address: string): boolean;
  normalise(tx: CardanoTransaction, address: string): NormalisedActivity[];
  getYieldStats?(
    address: string,
    stakeAddress: string | null,
    blockfrostKey: string,
  ): Promise<ProtocolYieldStats>;
};
```

Detection is mostly transaction-evidence based: known script addresses, redeemer script hashes, reference script hashes, mint/burn policies, wallet asset deltas, and protocol-specific fallback heuristics. Treat Cardamom output as indexed intelligence, not as the protocol ABI.

## Data providers

- Chain provider: Blockfrost.
- SundaeSwap positions: SundaeSwap GraphQL portfolio endpoint.
- Minswap positions: wallet LP token scan plus Minswap pool API enrichment.
- Liqwid positions: wallet qToken scan plus Liqwid market API enrichment.
- Indigo positions: `@indigo-labs/indigo-sdk` datum parsing plus Blockfrost script UTxO scans.
- FluidTokens, Dano, Surf positions: partial UTxO/token scans; no full redemption-value or health-factor decode.

## Cardamom-specific constants

These constants are useful for understanding Cardamom's current adapter behavior. They are not protocol authority; prefer each protocol's own docs/source before building a write-path integration.

| Protocol | Cardamom signal |
|---|---|
| Minswap | LP policies: [`13aa2accf2e1561723aa26871e071fdf32c867cff7e7d50ad470d62f`](https://adastat.net/policies/13aa2accf2e1561723aa26871e071fdf32c867cff7e7d50ad470d62f), [`f5808c2c990d86da54bfc97d89cee6efa20cd8461616359478d96b4`](https://adastat.net/policies/f5808c2c990d86da54bfc97d89cee6efa20cd8461616359478d96b4) |
| SundaeSwap | V3 LP policy: [`4de79a0c17180030bff4c36825cb6e99caa007bc632f789561a26d56`](https://adastat.net/policies/4de79a0c17180030bff4c36825cb6e99caa007bc632f789561a26d56) |
| FluidTokens | V3 config NFT policy: [`219832152b2c489358f4c02a1818d312a851b1f55774ae881e33a907`](https://adastat.net/policies/219832152b2c489358f4c02a1818d312a851b1f55774ae881e33a907), config address [`addr1wysesvs49vky3y6c7nqz5xqc6vf2s5d374thft5grce6jpcwela6v`](https://adastat.net/addresses/addr1wysesvs49vky3y6c7nqz5xqc6vf2s5d374thft5grce6jpcwela6v) |
| Liqwid | qToken policy: [`da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24`](https://adastat.net/policies/da8c30857834c6ae7203935b89278c532b3995245295456f993e1d24) |
| Indigo | CDP/staking validator address set from `../cardamom/lib/protocols/indigo.ts`; parsed with Indigo SDK when available |
| Dano | CLMM LP policy [`d8b69fc53637bcfadbc4469083f706bc293f4d9d2296646c5ca167bb`](https://adastat.net/policies/d8b69fc53637bcfadbc4469083f706bc293f4d9d2296646c5ca167bb), lending pool NFT policy [`814de8a99452972a9fa9fe2c0f59f49697f208005c001ecac1ddfd57`](https://adastat.net/policies/814de8a99452972a9fa9fe2c0f59f49697f208005c001ecac1ddfd57), dADA policy [`94dca24a1f1fcc2ff51cd90f32f4fe9e786d861a2dbf7d27598d26e8`](https://adastat.net/policies/94dca24a1f1fcc2ff51cd90f32f4fe9e786d861a2dbf7d27598d26e8) |
| Surf | COCK/ADA receipt policy [`daa59d07e4c35a2b32c36017534b3346e0576968d45d5b0e5cf41436`](https://adastat.net/policies/daa59d07e4c35a2b32c36017534b3346e0576968d45d5b0e5cf41436), pool NFT policy [`b6cd0e95575ffa5d9efd08a0d06961cf6dfd806a7104ea90b34dd18`](https://adastat.net/policies/b6cd0e95575ffa5d9efd08a0d06961cf6dfd806a7104ea90b34dd18) |

## Gotchas

- Cardamom frequently classifies by evidence, not full datum decode. Confidence levels matter.
- Blockfrost `/txs/{hash}/mints` can fail for Plutus-script minting transactions; several adapters fall back to redeemers or UTxO deltas.
- Some position endpoints return raw receipt/LP token balances when protocol APIs or SDKs cannot produce live redemption values.
