# v0.2.2-dev: Fungible Token Transfers

Status: ✅ Done  
Files:
- [crypto_transfer_transaction.dart](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/lib/src/transactions/crypto_transfer_transaction.dart)
- [crypto_transfer_transaction_test.dart](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/test/unit/transactions/crypto_transfer_transaction_test.dart)
- [token_transfer_example.dart](https://github.com/nemorixgroup/hedera-flutter-sdk/blob/main/example/phase3/token_transfer_example.dart)

## Summary

Extends `CryptoTransferTransaction` with `addTokenTransfer()`,
enabling fungible token transfers between accounts, alongside the
existing HBAR transfer support.

## Files Added / Modified

### Added

- `example/phase3/token_transfer_example.dart`; live testnet example
- `test/unit/transactions/crypto_transfer_transaction_test.dart`;
  18 new unit tests (token transfer setters, validation,
  serialization, `buildBody()` integration)

### Modified

- `lib/src/transactions/crypto_transfer_transaction.dart`;
  added `addTokenTransfer()`, `tokenTransferCount`, and extracted
  `_buildCryptoTransferBody()` shared by `toBytes()`/`applyToBody()`

## Official References

- https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service/transfer-tokens

## Key Decisions

### No new transaction class

Hedera has no dedicated token transfer transaction. HBAR, fungible
tokens, and NFTs are all transferred through the same
`CryptoTransferTransaction`/`TransferTransaction`, via
`addHbarTransfer()`, `addTokenTransfer()`, and `addNftTransfer()`.
This version extends the SDK's existing `CryptoTransferTransaction`
class rather than introducing a new one, matching that shape.

### Per-token grouping and independent sum-to-zero validation

Transfers are grouped internally by `TokenId`
(`Map<TokenId, List<_TokenAmount>>`), since each distinct token
becomes its own `TokenTransferList` in the Protobuf body. Each
token's amounts are validated to sum to zero independently from
HBAR amounts and from other tokens, matching the official
requirement that the sum of transfers within each
`TokenTransferList` must be zero.

### expectedDecimals is per-token, not per-entry

`expectedDecimals` (`UInt32Value`) is a field on `TokenTransferList`
itself, confirmed against `basic_types.pb.dart`, not on each
individual `AccountAmount` entry. If `addTokenTransfer()` is called
multiple times for the same token with conflicting non-null
`expectedDecimals` values, the SDK throws `ArgumentError` locally
rather than silently choosing one. No official documentation was
found addressing this specific edge case; the stricter behavior was
chosen deliberately, consistent with the SDK's general preference
for explicit errors over silent guesses.

### NFT transfers deferred

`TokenTransferList` also has an `nftTransfers` field, confirmed in
the same review, but NFT support is not implemented until
v0.2.4-dev, so `addNftTransfer()` is intentionally left out of this
version.

### AccountBalanceQuery limitation discovered

While building the live example, `AccountBalanceQuery` (built in
Phase 2, before tokens existed) was found to only expose HBAR
balance, with no per-token balance data. The example works around
this by tracking balances locally as running totals from each known
transfer. Extending `AccountBalanceQuery` with per-token balances is
planned for v0.2.6-dev (token queries).

## Verified

Live on Hedera testnet:

- Created a treasury account and a fungible token ("Demo Coin" /
  DEMO)
- Created and associated a second account (Bob)
- Transferred 25.00 DEMO treasury -> Bob (`SUCCESS`)
- Transferred 10.00 DEMO Bob -> treasury (`SUCCESS`)
- Transferred 1.00 DEMO with `expectedDecimals` set correctly
  (`SUCCESS`)
- Confirmed a conflicting `expectedDecimals` call across two
  `addTokenTransfer()` calls for the same token is caught locally,
  with no network call

## Tests

18 unit tests added to `crypto_transfer_transaction_test.dart`:
token transfer setters, per-token sum-to-zero validation,
`expectedDecimals` conflict detection, serialization, and
`buildBody()` integration.

## Related

- [v0.2.1-dev: Account-to-Token Association](https://github.com/nemorixgroup/Hedera-Knowledge-Base/blob/main/docs-sdk/phase-3/v0.2.1-dev/README.md)
