# v0.2.3-dev: Fungible Token Mint / Burn

Status: ✅  Done

## Summary

Implements `TokenMintTransaction` and `TokenBurnTransaction` for
fungible token supply management. Also fixes an intermittent
transaction reliability issue discovered while verifying this
version live, affecting every transaction type in the SDK.

## Files Added / Modified

### Added

- `lib/src/transactions/tokens/token_mint_transaction.dart`;
  `TokenMintTransaction`
- `lib/src/transactions/tokens/token_burn_transaction.dart`;
  `TokenBurnTransaction`
- `example/phase3/token_mint_burn_example.dart`; live testnet example
- `test/unit/transactions/tokens/token_mint_transaction_test.dart`;
  `test/unit/transactions/tokens/token_burn_transaction_test.dart`;
  28 unit tests total

### Modified

- `lib/src/transactions/transaction.dart`; `buildBody()` now
  subtracts a 5-second safety margin when building the transaction
  valid start time (see Key Decisions)
- `lib/src/transactions/token_create_transaction.dart`,
  `token_associate_transaction.dart`,
  `token_dissociate_transaction.dart`; moved into
  `lib/src/transactions/tokens/` (no content changes, only location)
- `lib/hedera_flutter_sdk.dart`; updated export paths for the moved
  files, added exports for the two new classes

## Official References

- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/mint-a-token
- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/burn-a-token

## Key Decisions

### Folder reorganization: transactions/tokens/

With this version, token-related transaction classes reached five
(`create`, `associate`, `dissociate`, `mint`, `burn`), with several
more still planned (NFT, KYC, freeze, queries, custom fees). Rather
than wait until `transactions/` became difficult to navigate, the
existing token classes were moved into a `tokens/` subfolder now,
alongside the two new ones, with tests mirrored the same way. This
has no effect on the SDK's public API, since every class is still
exported from the same barrel file; only internal file locations and
their export paths changed.

### Fungible-only scope, NFT deferred

Both classes expose only `setTokenId()` and `setAmount()`. NFT
support (`metadata` for mint, `serialNumbers` for burn) is deferred
to v0.2.4-dev, consistent with the same fungible-first, NFT-later
pattern already used for `CryptoTransferTransaction.addTokenTransfer()`
in v0.2.2-dev.

Verifying `TokenBurnTransactionBody` against the generated
`token_burn.pb.dart` surfaced another proto-name mismatch: the
documented `setSerials()`/`addSerial()` methods correspond to a
`serialNumbers` field, not `serials`.

### setHighVolume() still unresolved

The official mint documentation explicitly lists `setHighVolume()`
(HIP-1313) as a `TokenMintTransaction` property, with a dedicated
explanation of high-volume throttle routing. The burn documentation
does not mention it at all. Checking the actual generated protobufs
confirmed neither `TokenMintTransactionBody` nor
`TokenBurnTransactionBody` has a `highVolume` field. This is the
same open question first raised in v0.2.1-dev for
`TokenAssociateTransaction`: HIP-1313's rollout appears to be
per-transaction-type, and documentation for some transactions is
ahead of the protobufs actually generated so far.

### SDK-wide fix: transaction valid start time safety margin

While verifying this version live, `TokenCreateTransaction` failed
intermittently with `INVALID_TRANSACTION_START` (response code 5),
even with the client's clock confirmed accurate to within 0.2
seconds via an external time source. `Transaction.buildBody()` built
the valid start timestamp directly from `DateTime.now()`, with no
margin for the receiving consensus node's own clock lagging slightly
behind the client's.

Confirmed via a real issue in the official Java SDK
(`hiero-ledger/hiero-sdk-java#1652`) that earlier SDK versions
subtracted 5 to 8 seconds from the client's clock specifically to
guard against this, and that removing that margin reintroduced the
exact same failure mode. Fixed by subtracting a 5-second margin when
building the timestamp. This fix is not scoped to mint/burn; it
applies to every transaction type built by the SDK, including
already-published versions.

## Verified

Live on Hedera testnet: created a token with a supply key, minted
50.00 DEMO (`SUCCESS`, supply 100.00 to 150.00), burned 30.00 DEMO
(`SUCCESS`, supply 150.00 to 120.00), confirmed on HashScan.

## Tests

28 unit tests across `token_mint_transaction_test.dart` and
`token_burn_transaction_test.dart`: defaults, setters, `toBytes()`
validation and serialization (including confirming `metadata`/
`serialNumbers` stay empty since NFT support is deferred), and
`buildBody()` integration confirming `tokenMint`/`tokenBurn` oneof
field routing.

## Related

- [v0.2.2-dev: Fungible Token Transfers](../v0.2.2-dev/README.md)
