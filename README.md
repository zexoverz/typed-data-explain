# typed-data-explain

Turn an EIP-712 signature request into one plain sentence and a list of risks, before anyone signs it.

Most wallet drains today do not steal a key. They get one signature on typed data the user could not
read: a `Permit` with an unlimited value, a `Permit2` batch for every token in the wallet, a
`TransferWithAuthorization` valid for a year, a Seaport order selling an NFT for nothing. The JSON is
right there, but nobody can read it, and the libraries on npm only hash it.

## Planned API

```ts
import { explainTypedData } from "typed-data-explain";

const result = explainTypedData(request, { chainId: 8453, now: Date.now() / 1000 });
// result.summary: "Lets 0x1111…3333 spend unlimited USDC from your wallet until 2027-09-27."
// result.kind:    "erc2612-permit" | "permit2-single" | "permit2-batch" | "eip3009-transfer" | "seaport-order" | "unknown"
// result.risks:   [{ code: "unlimited-amount" | "far-deadline" | "chain-mismatch" | "zero-consideration" | "unknown-verifying-contract", severity, detail }]
```

- Recognizes ERC-2612 `Permit`, Uniswap `Permit2` (`PermitSingle`, `PermitBatch`, `PermitTransferFrom`),
  EIP-3009 `TransferWithAuthorization` / `ReceiveWithAuthorization` and Seaport `OrderComponents`.
- Validates the request shape with zod and checks the domain against the wallet's chain.
- Amounts rendered with token decimals when the caller passes token metadata; never fetches on its own.
- Unknown types still get a generic, honest summary instead of a guess.

Built at ETHGlobal Tokyo 2026 as a working project of the End Credits demo: the library is written in
a Claude Code session, and End Credits pays the open-source packages that session used.

## Demo fixtures

The dev dependencies under `@endcredits-demo/*` are End Credits demo fixtures, not real libraries.
They exist so one session can show every outcome End Credits handles, without ever showing a real
package as suspicious:

| Fixture | Shows |
|---|---|
| `moved-payout` | a payout address that changed: held until the owner signs |
| `left-padder-pro` | a sanctioned address: refused by Intercepta |
| `unclaimed-utils` | no wallet yet: reserved until the maintainer claims |
| `tip-jar` | paid through the maintainer's own x402 endpoint |
| `swapped-jar` | an endpoint that asks to be paid somewhere else: refused before signing |

## License

MIT
